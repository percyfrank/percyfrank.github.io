---
title: "JPA Batch Insert 적용하기"
excerpt: "JPA IDENTITY 전략은 Batch Insert를 사용할 수 없다."
categories:
  - Spring Data
tags:
  - Spring Data
date: 2022-02-25
last_modified_at: 2022-02-25
---

## 1. Batch(Bulk) Insert

> SQL

```sql
# 단건 Insert
insert into user (id, name) values (1, 'a');
insert into user (id, name) values (2, 'b');
insert into user (id, name) values (3, 'c');
insert into user (id, name) values (4, 'd');

# Batch Insert
insert into payment_back (amount, order_id)
values (1, 'a'), (2, 'b'), (3, 'c'), (4, d);
```

Batch(Bulk) Insert란 복수의 SQL Statement 행을 단건의 구문으로 처리하는 방식이다. DB로 향하는 네트워크 호출 횟수를 N건에서 1건으로 줄인다. 일괄 작업이라고 번역할 수 있겠다. 1000개의 데이터를 DB에 저장해야 할 때, 어플리케이션에서 네트워크를 통해 쿼리를 보내 DB 쓰기 연산을 수행하는 행위를 1000번 반복하는 것 보다는 1건으로 합쳐서 수행하는 것이 효율적임이 자명하다.

CSV 파일을 DB에 저장하는 API를 개발하면서 자연스럽게 Bulk Insert 적용을 고민하게 되었다. CSV 파일에 영속화할 데이터가 많아질수록, 루프를 돌며 개별적으로 엔티티를 저장하는 로직의 효율성이 떨어지기 때문이다. 아무튼 JPA를 사용하는 프로젝트에서 Batch Insert를 적용해보자.

<br>

## 2. 설정

### 2.1. Hibernate

[Hibernate Reference 12. Batching](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch)을 참고해 JPA 상에서 Batch Insert를 적용하기 위한 관련 설정들을 찾아보았다.

> hibernate.jdbc.batch_size

DB 드라이버에게 Batch를 수행하도록 요청하기 전, Hibernate가 함께 Batch를 처리할 구문의 최대 개수를 지정한다. 즉, 일괄적으로 처리해 DB로 보낼 구문의 최대 개수를 의미한다. 0 혹은 음수는 해당 기능을 비활성화한다.

> hibernate.order_updates

SQL Update 구문의 순서를 업데이트 되는 아이템의 엔티티 타입 및 PK 값을 기준으로 정렬하도록 한다. 더 많은 Batch 작업이 수행될 수 있도록 한다. 동시성 수준이 높은 시스템에서 트랜잭션의 데드락을 거의 유발하지 않는다. 실제로 어플리케이션에 도움이 되는지 성능 측정할 것을 권고한다.

> hibernate.order_inserts

SQL Insert 구문의 순서를 정렬해 더 많은 Batch 작업이 수행될 수 있도록 한다. 마찬가지로 어플리케이션에 도움이 되는지 성능 측정할 것을 권고한다.

> SQL

```sql
# Order 옵션 적용 전
update a set name = '1' where id = 1;
update b set name = '2' where id = 2;
update a set name = '3' where id = 3;
update b set name = '4' where id = 4;


# Order 옵션 적용 후
update a set name = '1' where id = 1;
update a set name = '3' where id = 3;
update b set name = '2' where id = 2;
update b set name = '4' where id = 4;
```

Order 관련 설정은 Batch 작업을 더 효율적으로 사용할 수 있는 옵션이라고 이해하면 좋다. 하나의 트랜잭션에서 여러 테이블에 대해 쓰기 작업이 진행될 때, 설정 적용 전에는 발생한 순서대로 쿼리가 생성된다. 그러나 옵션을 적용하면 같은 테이블의 구문끼리 정렬해 쿼리가 수행되도록 한다. 즉, Batch 작업시 일괄 처리되는 숫자가 많아지는 효과를 기대할 수 있다.

### 2.2. MySQL

[MySQL 8.0 Reference 6.3.13. Performance Extensions](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-performance-extensions.htmll)에 따르면 ``rewriteBatchedStatements`` 옵션을 true로 주어야 여러 쓰기 쿼리를 단건으로 합친다고 한다.

### 2.3. YML 설정 파일

> application.yml

```yml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        dialects: org.hibernate.dialect.MySQL57Dialect
        format_sql: true
        jdbc:
          batch_size: 1000
        order_inserts: true
        order_updates: true
    generate-ddl: true
    hibernate:
      ddl-auto: create-drop
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/batch_insert?serverTimezone=UTC&characterEncoding=UTF-8&rewriteBatchedStatements=true&profileSQL=true&logger=Slf4JLogger&maxQuerySizeToLog=999999
    username: root
    password: root
    hikari:
      data-source-properties:
        rewriteBatchedStatements: true
```

> DB URL에서도 rewriteBatchedStatements=true 옵션을 잊지 말자.

실제 Batch Insert가 이뤄지는지 확인하기 위해서는 Hibernate의 ``show-sql`` 옵션으로는 충분하지 않다. Hibernate의 로그는 Hibernate가 DB 드라이버에게 보낸 내용만을 출력한다. Hibernate는 DB 드라이버로 여러 개의 Insert 쿼리를 보내며 Batch Insert 관련 쿼리는 출력하지 않는다. 다음 옵션을 통해 실제 DB 드라이버가 DB로 보내는 Batch Insert 쿼리를 확인한다.

* profileSQL : Driver에서 전송하는 쿼리를 출력한다.
* logger : Driver에서 쿼리 출력시 사용할 Logger를 설정한다.
* maxQuerySizeToLog : 출력할 쿼리 길이를 설정한다.

<br>

## 3. Hibernate의 제약 사항

> Hibernate disables insert batching at the JDBC level transparently if you use an identity identifier generator.

[Hibernate Reference 12.2. Session Batching](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch)에 다음과 같은 제약 사항이 존재한다. **식별자 생성 전략으로 IDENTITY를 사용하면 Hibernate는 JDBC 레벨의 Batch Insert를 비활성화한다.**

JPA의 영속성 컨텍스트는 쓰기 지연 저장소를 제공하며, 트랜잭션 커밋 시점에 ``em.flush()``를 호출해 영속성 컨텍스트와 DB의 동기화가 이루어진다. Batch Insert는 트랜잭션 커밋 시점에 지연된 쿼리들을 Batch 처리하는 것이 골자다. 문제는 DB를 통해 PK를 증분하는 식별자 생성 전략은 실제 데이터를 Insert하기 전 까지 ID를 알 수가 없다.

IDENTITY 전략을 사용할 때 ID가 설정되지 않은 엔티티에 대해 ``jpaRepository.save()``를 호출하면, 영속성 컨텍스트는 ``em.persist()`` 이후 ID를 획득하기 위해 Insert 쿼리를 쓰기 지연하지 않고 실행한다. 이 때 채번된 ID를 포함한 엔티티 객체가 영속성 컨텍스트에서 영속 상태로 관리된다. 따라서 IDENTITY 전략을 사용하면 쓰기 지연이 되지 않아 Batch Insert가 불가능하다.

* 반면 영속화하려는 엔티티 객체에 식별자가 존재하는 경우 ``jpaRepository.save()``는 Select 쿼리를 먼저 보낸다.
  * 해당 식별자의 엔티티가 존재하지 않으면 Insert 쿼리를 생성해 쓰기 지연 저장소에 저장한다.
  * 해당 식별자의 엔티티가 존재하면 이를 영속성 컨텍스트의 1차 캐시에 저장하고, ``jpaRepository.save()`` 파라미터로 들어온 동일 식별자의 엔티티에 대해 내부적으로 ``em.merge()``를 수행한다.
    * 트랜잭션 커밋 시점에 Update 쿼리를 수행한다.
* ``em.merge()``의 플로우는 다음과 같다.
  * 파라미터로 주어진 준영속 엔티티 식별자와 동일한 엔티티를 1차 캐시를 조회하고, 없는 경우 DB에서 조회해 1차 캐시에 저장한다.
  * 조회된 영속 엔티티의 속성 값을 파라미터로 주어진 준영속 엔티티의 값으로 변경한다.
  * 트랜잭션 커밋 시점에 더티 체킹이 발생해 Update 쿼리를 수행한다.

아무튼 다시 본론으로 돌아와서, Hibernate의 제약 사항을 회피하는 방법은 다음과 같다.

1. SEQUENCE(혹은 TABLE) 방식을 사용하면서 채번 부하를 낮추는 방법.
2. JPA를 사용하지 않고 Batch Insert를 사용하는 방법.

1번에 대해 간략히 짚고 넘어가겠다.

### 3.1. SEQUENCE(TABLE) 채번 방식

MySQL은 SEQUENCE를 제공하지 않으니 별도의 채번 Table을 사용한다. 동작 방식은 다음과 같다.

1. ``jpaRepository.save()``를 호출하면 ``em.persist()``는 채번 테이블에서 SEQUENCE 번호를 가져온다.
2. 가져온 SEQUENCE 번호를 영속성 컨텍스트에서 영속 상태로 관리되는 엔티티의 ID에 할당하고, 트랜잭션이 커밋되는 시점에 쓰기 지연 Insert 쿼리가 나간다.

즉, SEQUENCE 전략을 사용하면 Batch Insert의 사용이 가능하다. 다만 SEQUENCE 테이블로 인한 성능 저하가 발생할 수 있다.

> SQL

```sql
...
SET autocommit=0
select next_val as id_val from hibernate_sequence for update
update hibernate_sequence set next_val= 2 where next_val=1
commit
autocommit=1
autocommit=0
select next_val as id_val from hibernate_sequence for update
update hibernate_sequence set next_val= 3 where next_val=2
commit
SET autocommit=1
SET autocommit=0
select next_val as id_val from hibernate_sequence for update
update hibernate_sequence set next_val= 4 where next_val=3
commit
SET autocommit=1
...
```

SEQUENCE 번호를 한 번씩 얻을 때마다 추가 쿼리가 2개씩 발생한다. 다행히 SEQUENCE 번호 채번 또한 낱개 단위가 아닌 Batch를 통해 복수개를 가져올 수 있다. 이를 통해 채번 쿼리 횟수를 낮춰 성능을 대폭 향상시킬 수 있다.

그러나 다음과 같은 단점이 존재한다.

* Batch 채번을 위해 다소 복잡한 Hibernate 전용 애너테이션을 지정해야 한다.
* 채번 Batch 크기 또한 애너테이션 내에서 지정하므로, Batch 크기 설정을 yml 파일로 외부화할 수 없다.
  * 반면 Batch Insert Batch 크기는 yml 파일로 지정하므로 두 값이 달라질 가능성이 존재한다.

SEQUENCE 전략은 [Spring Data에서 Batch Insert 최적화](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/) 글을 참고하자.

<br>

## 4. 성능 비교

> BatchInsertTest.java

```java
@Test
void batchInsert() {
    List<User> list = new ArrayList<>(100_000);

    for (int i = 1; i <= 100_000; i++) {
        list.add(new User(null, 1L, 1L, "hi", 10000L, LocalDate.now()));
    }

    StopWatch stopWatch = new StopWatch();
    stopWatch.start();

    service.addAll(list);

    stopWatch.stop();
    System.out.println(stopWatch.prettyPrint());

    assertThat(UserRepository.count()).isEqualTo(100_000);
}
```

> Service.java

```java
@Transactional
public void addAll(List<User> ts) {
    UserRepository.saveAll(ts);
}
```

* 엔티티 10만개를 Batch Insert하는 테스트를 통해 각 방식의 성능을 비교해본다.
* Batch 크기는 1000개로 한다.

### 4.1. JPA IDENTITY 전략

> User.java

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    ...
}
```

* 쓰기 지연이 동작하지 않아 Batch Insert를 적용할 수 없다.

<img width="339" alt="image" src="https://user-images.githubusercontent.com/56240505/155673271-af391637-f543-4eba-a046-e64d82afc1a9.png">

* Insert 쿼리가 10만개 발생한다.
* 116.8초, 대략 2분이 소요된다.

### 4.2. JPA 식별자 수동 생성 전략

> User.java

```java
@Entity
public class User {

    @Id
    private Long id;

    ...
}
```

> BatchInsertTest.java

```java
@Test
void batchInsert() {
    List<User> list = new ArrayList<>(100_000);

    for (int i = 1; i <= 100_000; i++) {
        list.add(new User((long) i, 1L, 1L, "hi", 10000L, LocalDate.now()));
    }

    ...
}
```

* Batch Insert를 적용하기 위해 식별자를 수동으로 생성해 넣도록 한다.

<img width="346" alt="image" src="https://user-images.githubusercontent.com/56240505/155676704-4d8e2bac-0726-40b4-9937-e27d8a2c2922.png">

* 120.36초, 대략 2분이 소요된다.

성능이 나쁜 이유는 Batch Insert 설정과 관계없이 JPA 영속성 컨텍스트의 동작 원리 때문이었다. 쿼리 로그를 보니 DB는 정상적으로 Batch Insert 쿼리를 수신 중이었다. 병목의 원인은 Select 쿼리 10만개였다.

앞서 언급했듯, 영속화하려는 엔티티 객체에 식별자가 존재하는 경우 ``jpaRepository.save()``는 Select 쿼리를 먼저 보낸다. 해당 식별자의 엔티티가 존재하지 않으면 Insert 쿼리를 생성해 쓰기 지연 저장소에 저장한다. 해당 식별자의 엔티티가 존재한다면 영속성 컨텍스트의 1차 캐시에 저장하고, 해당 엔티티의 속성 값을 ``em.merge()`` 를 통해 파라미터로 주어진 준영속 엔티티의 값으로 변경한다. 따라서 트랜잭션 커밋 시점에 더티 체킹이 발생해 Update 쿼리가 발생한다.

이전 절의 IDENTITY 전략은 쓰기 지연이 불가능해 10만개의 Insert 쿼리가 발생해 병목이 생겼다. **이번 절의 전략은 쓰기 지연이 가능하며, 실제로 DB 쿼리 로그를 보면 Batch Insert가 정상 동작한다. 그러나 JPA 영속성 컨텍스트의 동작 방식으로 인해, 트랜잭션에서 Select 쿼리가 10만개 발생한다.** 결과적으로는 이전 절의 IDENTITY 전략과 동일하게 병목으로 인한 성능 저하가 발생한다.

### 4.3. JPA SEQUENCE 전략

> User.java

```java
@TableGenerator(
    name = "SequenceGenerator",
    table = "MY_SEQUENCE",
    pkColumnName = "sequence_name",
    valueColumnName = "next_val",
    allocationSize = 1000
)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class User {

    @Id
    @GeneratedValue(
        strategy = GenerationType.TABLE,
        generator = "SequenceGenerator"
    )
    private Long id;

    ...
}
```

* Batch Insert를 적용하기 위해 SEQUENCE 전략을 사용한다.
  * MySQL은 SEQUENCE를 지원하지 않아 예제는 TABLE 전략을 사용했다.
* 채번 테이블 또한 마찬가지로 Batch 사이즈를 1000으로 지정해준다.

<img width="346" alt="image" src="https://user-images.githubusercontent.com/56240505/155673735-ff613fab-8e78-48ba-90fe-9c92ed063ed7.png">

* 7.01초가 소요된다.

### 4.4. JDBC BatchUpdate

> Dao.java

```java
public void add(List<User> ts) {
    String sql =
        "insert into user (user_id, height, weight, description, money, date) values(?, ?, ?, ?, ?, ?)";
    jdbcTemplate.batchUpdate(
        sql, ts, 1000, (ps, argument) -> {
            ps.setLong(1, argument.getUserId());
            ps.setLong(2, argument.getHeight());
            ps.setString(3, argument.getWeight());
            ps.setString(4, argument.getDescription());
            ps.setDate(5, argument.getDate().toString());
        });
}
```

* 테이블의 PK가 AUTO_INCREMENT로 자동 증분될 때, JPA가 아닌 Spring Data JDBC의 ``batchUpdate``로 Batch Insert하도록 적용해보았다.
* 예전에 정리해둔 [Spring Data JDBC의 batchUpdate](https://xlffm3.github.io/spring%20data/jdbc-batch-update/) 글을 참고했다.

<img width="335" alt="image" src="https://user-images.githubusercontent.com/56240505/155674704-248516e6-2c7d-4dbc-b9b2-b5ecb9566fca.png">

* 약 4.3초가 소요된다.

<br>

## 5. 마치며

측정 결과, Batch Insert 성능은 Spring Data JDBC가 Spring Data JPA보다 우수하다. 이와 관련해 참고할만한 블로그 글을 첨부한다.

> MySQL에는 Sequence가 없으므로 SEQUENCE 방식을 지정했다고 하더라도 사실상 TABLE 방식으로 동작했다는 것을 감안하면, Sequence가 지원되는 DB에서는 TABLE 방식보다 채번 부하가 더 적은 SEQUENCE 방식을 Batch 스타일로 사용하면 Spring Data JDBC 방식과 비슷한 성능을 보일 것 같다.

정말 많은 데이터를 저장해야 한다면 JPA를 고수할 필요는 없어보인다.

<br>

---

## References

* [Hibernate Reference 12. Batching](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch)
* [Hibernate Reference 12.2. Session Batching](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch)
* [MySQL 8.0 Reference 6.3.13. Performance Extensions](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-connp-props-performance-extensions.htmll)
* [Spring Data에서 Batch Insert 최적화](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/)
* [Spring Boot, JPA - Batch Insert](https://blog.naver.com/PostView.naver?blogId=hsy2569&logNo=222151651874)
