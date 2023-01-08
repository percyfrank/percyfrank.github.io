---
title: "InitializingBean 및 EntityManager를 활용한 테스트 격리"
excerpt: "@Sql 및 @DirtiesContext 이외의 방안에 대해 알아보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-08-25
last_modified_at: 2021-08-25
---

## 1. 테스트 격리

Spring Boot Test의 경우 기본적으로 최초에 생성한 Bean Container(ApplicationContext)를 재활용한다. 테스트가 진행됨에 따라서 저장된 데이터의 개수 등 DB 상태는 계속 변경된다.

따라서 단독으로 돌릴 때는 잘 동작하던 테스트가, ``Run All Tests``와 같이 모든 테스트를 한번에 실행할 때 실패하는 경우가 종종 발생한다. 전위의 테스트 실행 결과가 후위의 테스트에 영향을 주는 등 테스트들이 완벽하게 격리되지 않았기 때문이다. 테스트의 독립성을 유지하기 위해 테스트를 격리시켜야 한다.

### 1.1. 격리 방법

* @AfterEach로 매 테스트 수행 전후로 데이터를 삭제함으로써 DB를 원상복구시킨다.
  * 삭제해야 할 데이터가 많으면 테스트 수행 속도가 느려진다.
  * 삭제해야 할 데이터가 복잡한 연관 관계 맵핑을 가지고 있다면, 관련 엔티티 데이터까지 삭제해야한다.
    * 외래키 참조 무결성 제약 등의 에러가 발생하기 쉽고, 도메인 지식이 없다면 삭제 작업을 작성하기 어렵다.
* @Sql 애너테이션을 통해 매 테스트 수행 전후로 미리 준비된 Truncate SQL을 실행한다.
  * 엔티티 스키마에 변경이 발생할 때마다 함께 수정되어야 하는 번거로움이 존재한다.
* @DirtiesContext를 활용한다.
  * 매 테스트 수행 전후로 Context를 다시 로드하는 만큼, 테스트 시간이 오래 걸린다.

<br>

## 2. InitializingBean 및 EntityManager 활용

> DataBaseCleaner.java

```java
@Component
@ActiveProfiles("test")
public class DatabaseCleaner implements InitializingBean {

    @PersistenceContext
    private EntityManager entityManager;

    private List<String> tableNames;

    @Override
    public void afterPropertiesSet() {
        entityManager.unwrap(Session.class)
            .doWork(this::extractTableNames);
    }

    private void extractTableNames(Connection conn) throws SQLException {
        List<String> tableNames = new ArrayList<>();

        ResultSet tables = conn
            .getMetaData()
            .getTables(conn.getCatalog(), null, "%", new String[]{"TABLE"});

        while(tables.next()) {
            tableNames.add(tables.getString("table_name"));
        }

        this.tableNames = tableNames;
    }

    public void execute() {
        entityManager.unwrap(Session.class)
            .doWork(this::cleanUpDatabase);
    }

    private void cleanUpDatabase(Connection conn) throws SQLException {
        Statement statement = conn.createStatement();
        statement.executeUpdate("SET REFERENTIAL_INTEGRITY FALSE");

        for (String tableName : tableNames) {
            statement.executeUpdate("TRUNCATE TABLE " + tableName);
            statement.executeUpdate("ALTER TABLE " + tableName + " ALTER COLUMN ID RESTART WITH 1");
        }

        statement.executeUpdate("SET REFERENTIAL_INTEGRITY TRUE");
    }
}
```

> AcceptanceTest.java

```java
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class AcceptanceTest {

    @Autowired
    private WebTestClient webTestClient;

    @Autowired
    private DatabaseCleaner databaseCleaner;

    @AfterEach
    void tearDown() {
        databaseCleaner.execute();
    }

    //...
}
```

* 미리 정의된 SQL 파일을 실행하기 보다는, EntityManager를 통해 모든 테이블 이름을 조사해서 테스트 전후로 TRUNCATE 쿼리를 실행하는 방식이다.

<br>

---

## References

* [인수테스트에서 테스트 격리하기](https://tecoble.techcourse.co.kr/post/2020-09-15-test-isolation/)
