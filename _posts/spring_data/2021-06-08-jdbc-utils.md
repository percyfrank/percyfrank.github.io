---
title: "Spring Data JDBC 유틸리티 클래스"
excerpt: "DataAccessUtils 및 DataSourceUtils 등 헬퍼 클래스에 대해 알아보자."
categories:
  - Spring Data
tags:
  - Spring Data
toc: true
toc_sticky: true
last_modified_at: 2021-06-08
---

## 1. DataAccessUtils

DAO 구현시 사용할 수 있는 유틸리티 클래스다. ``org.springframework.dao.support.DataAccessUtils``에 정의되어 있다. 세부적인 메서드 목록은 레퍼런스를 참고하자. 유틸리티 메서드 명이 직관적이라 부수적인 설명은 필요없을 것으로 사료된다.

* ``requiredSingleResult()`` 및 ``singleResult()``는 쿼리 수행 결과로 반환된 컬렉션의 크기가 1이면 해당 값을 반환하고, 아니면 예외를 뱉는다.
* ``translateIfNecessary()``는 인자로 들어온 RuntimeException이 적절한 경우 DataAccessException으로 번역해 반환하고, 그 외 주어진 예외를 그대로 반환한다.

<br>

## 2. DataSourceUtils

DataSource로부터 JDBC 커넥션을 획득하는 정적 메서드를 제공하는 헬퍼 클래스다. DataSourceTransactionManager 혹은 JtaTransactionManager와 같은 Spring 기능에 의해 관리되는 Transactional Connection에 대해 특별 지원이 포함되어 있다.

JdbcTemplate 및 Spring JDBC 작업 객체들이 내부적으로 사용하는 클래스이기도 하다. 응용 코드에서 직접 사용할 수 있다. ``org.springframework.jdbc.datasource.DataSourceUtils``에 정의되어 있다.

* ``getConnection()``은 JTA 트랜잭션 등 트랜잭션 동기화가 활성화되어 있을 때, 커넥션을 생성함과 동시에 동기화에 사용되도록 해당 커넥션을 트랜잭션 저장소(스레드)에 바인딩해준다.
* 트랜잭션 및 커넥션을 직접 제어해야 할 때 참고할만한 메서드들이 많이 정의되어 있다.

자세한 내용은 토비의 스프링 5장을 참고하자.

<br>

## 3. JdbcUtils

커스텀한 JDBC 액세스 코드 작성시 유용하게 사용할 수 있는 유틸리티 클래스다. ``org.springframework.jdbc.support.JdbcUtils``에 정의되어 있다. 커넥션을 직접 관리하는 코드 구조가 아니라면 사용할 일이 거의 없을듯 싶다.

<br>

---

## References

* [DataAccessUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/dao/support/DataAccessUtils.html)
* [DataSourceUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/DataSourceUtils.html)
* [JdbcUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/support/JdbcUtils.html)
