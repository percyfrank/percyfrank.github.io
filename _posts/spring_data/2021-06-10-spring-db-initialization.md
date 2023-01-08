---
title: "Spring Data JPA의 DB 초기화 전략"
excerpt: "DDL Auto 옵션에 대해 알아보자."
categories:
  - Spring Data
tags:
  - Spring Data
date: 2021-06-10
last_modified_at: 2021-06-10
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/jpa-ddl)

## 1. 서로 다른 DB 초기화

> application.properties

```java
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.h2.console.enabled=true
spring.jpa.show-sql=true

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
spring.jpa.properties.hibernate.dialect.storage_engine=innodb
spring.datasource.hikari.jdbc-url=jdbc:h2:mem://localhost/~/testdb;MODE=MYSQL
```

Spring Boot 프로젝트에 Spring Data JPA 및 H2 의존성을 추가해주고, 간단하게 H2 DB를 설정해준다.

* ``spring.jpa-show-sql`` 옵션은 JPA가 생성하는 SQL 쿼리를 보여준다.
* 콘솔에 찍히는 쿼리 로그를 H2가 아닌 MySQL 버전으로 보기 위해 방언을 추가 설정해준다.
  * 불필요하다면 하단의 설정 세 개를 삭제하고 ``spring.jpa.database-platform=org.hibernate.dialect.H2Dialect``만 선언하면 된다.

> User.java

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    //...
}
```

> Console

```
Hibernate: drop table if exists car
Hibernate: drop table if exists user
Hibernate: create table car (id bigint not null auto_increment, name varchar(255) not null, primary key (id)) engine=InnoDB
Hibernate: create table user (id bigint not null auto_increment, name varchar(255) not null, primary key (id)) engine=InnoDB
```

* 어플리케이션이 실행되면 @Entity에 해당하는 클래스들을 찾아 JPA가 자동으로 테이블을 생성해준다.

> Console

```
Hibernate: drop table if exists car
Hibernate: drop table if exists user
```

* 어플리케이션이 종료되면 생성해둔 테이블들을 전부 제거한다.

> application.properties

```java
spring.datasource.url=jdbc:mysql://localhost:13306/ddl
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root
spring.h2.console.enabled=true
spring.jpa.show-sql=true
```

* 이번에는 별도로 MySQL 서버를 띄워 연동시킨 뒤 어플리케이션을 구동한다.
* H2와는 다르게 자동으로 스키마를 생성하거나 삭제하지 않는다!

<br>

## 2. Hibernate 전략

H2 인메모리 DB를 사용하면 알아서 스키마를 초기화하지만, MySQL을 사용하면 초기화하지 않는다. 먼저 다음 옵션들을 알아야 한다.

* ``spring.jpa.generate-ddl`` : 어플리케이션 시작시 스키마 초기화 여부를 지정하며 기본값은 false다.
* ``spring.jpa.hibernate.ddl-auto`` : DB 초기화 전략 옵션이다.
  * ``none`` : DDL 핸들링 작업을 수행하지 않는다.
  * ``create`` : 기존에 존재하는 스키마를 삭제하고 새로 생성한다.
  * ``create-drop`` : 스키마를 생성하고 어플리케이션이 종료될 때 삭제한다.
  * ``update`` : 기존의 스키마를 유지하며 JPA에 의해 변경된 부분만 추가한다.
  * ``validate`` : 엔티티와 테이블이 정상적으로 매핑되어있는지만 검증한다.

> Spring Boot automatically creates the schema of an embedded DataSource.

인메모리 DB를 사용하면 별도의 옵션 설정을 하지 않더라도 자동으로 ``create-drop`` 전략을 채택하여 초기화하는 것을 알 수 있다. ``spring.jpa.generate-ddl``이 false이거나, true더라도 ``spring.jpa.hibernate-ddl-auto``를 별도로 지정하지 않는다면 항상 ``create-drop``이다.

반면 DB 벤더를 사용하는 경우 ``spring.jpa.generate-ddl=true``를 설정해야 초기화가 진행된다. 나의 경우, true일 때 ``spring.jpa.hibernate-ddl-auto`` 옵션을 별도로 지정하지 않아도 자동으로 ``create`` 전략을 채택하는 것을 확인했다. 단, 기본 초기화 전략은 DB 벤더 및 JPA 버전마다 상이하지 않을까 추측한다.

### 2.1. Update

> User.java

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String age;
}
```

* DB가 이미 초기화된 상태에서 User 클래스에 age 칼럼을 추가한 뒤, 어플리케이션을 다시 실행해본다.

> Console

```
Hibernate: alter table user add column age varchar(255) not null
```

* 변경된 스키마가 적용한다.

### 2.2. Validate

> User.java

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String age;

    @Column(nullabe = false)
    private String country;
}
```

* 테스트를 위해 칼럼을 하나 더 추가한 뒤, 어플리케이션을 실행해본다.

> Console

```
Schema-validation: missing column [country] in table [user]
```

* 엔티티와 테이블이 정상적으로 매핑되어있는지만 검증한다.
* 변경된 스키마가 있다면 변경점 에러를 출력하고 애플리케이션을 종료한다.

<br>

## 3. SQL Script를 통한 DB 초기화

> application.properties

```java
spring.datasource.initialization-mode=always
```

Script를 통해 DB를 초기화하려면 먼저 ``spring.jpa.generate-ddl`` 옵션을 삭제하거나 false로 둔다. Script 초기화 방식과 Hibernate DDL 전략 초기화 방식 모두를 사용할 수 없기 때문이다. 이후 ``spring.datasource.initialization-mode``를 활성화한다.

* 기본적으로 해당 옵션은 Embedded Datasource에 한해서만 초기화를 해주기 때문에, DataSource 타입에 상관없이 초기화하기 위해 ``always`` 옵션을 둔다.

Spring은 기본적으로 classpath에 schema.sql 파일이 있다면 서버 시작시 자동으로 실행한다. 보통 schema.sql은 DDL을 정의하고, data.sql이 DML을 정의해둔다.

* JPA는 schema-${platform}.sql 및 data-${platform}.sql 파일이 있으면 데이터베이스 플랫폼에 맞춘 스크립트 실행이 가능하다.
  * 사용할 플랫폼 정의는 ``spring.datasource.platform`` 값을 따른다.
* Hibernate는 classpath에 import.sql 파일이 있다면 서버 시작시 자동으로 실행한다.
  * Spring의 전략이 아니기 때문에, Spring이 관장하는 script.sql 등의 스크립트와 내용이 중복되면 모두 다 실행되면서 충돌이 발생할 수 있다.

<br>

---

## References

* [Database Initialization](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/howto-database-initialization.html)
* [스프링 부트 - 데이터베이스 초기화](https://kyu9341.github.io/java/2020/04/14/java_springBootDBinit/)
