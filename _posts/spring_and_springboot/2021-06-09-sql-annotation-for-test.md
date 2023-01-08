---
title: "@Sql 애너테이션을 활용한 테스트 격리"
excerpt: "테스트 독립성을 위해 데이터 초기화 용도로 사용된다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-09
last_modified_at: 2021-06-09
---

## 1. @Sql

테스트 클래스나 테스트 매서드에 부착하는 애너테이션이며, 지정된 특정 SQL 스크립트 혹은 Statement를 수행시킨다. 통합 테스트에서 편리하게 DB 스키마 생성과 초기 데이터 삽입 및 **데이터 초기화** 등을 수행할 수 있다.

대부분 테스트 독립성을 위해 데이터 초기화 용도로 사용된다. 매번 새로운 컨텍스트를 생성하는 @DirtiesContext를 사용하기 곤란한 경우 사용해보자!

### 1.1. 사용 예제

> SqlTest.java

```java
@Sql("classpath:/truncate.sql")
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = AppConfig.class)
public class SqlTest {
    //...
}
```

* 각각의 테스트 메서드 실행 전 @Sql 애너테이션이 지정한 스크립트를 실행한다.
* ``executionPhase``로 스크립트의 실행 시점을 조정할 수 있으며, 기본값은 BEFORE_TEST_METHOD다.
* ``@Sql({"a.sql", "b.sql", "c.sql"})``과 같이 복수의 스크립트 파일을 지정할 수 있다.

<br>

---

## References

* [Annotation Type Sql](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/jdbc/Sql.html)
