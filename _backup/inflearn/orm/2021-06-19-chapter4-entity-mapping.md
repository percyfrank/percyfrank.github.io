---
title: "[Java ORM 표준 JPA 프로그래밍] 4장 : 엔티티 매핑"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-19
last_modified_at: 2021-06-19
---

## 1. @Entity

* @Entity가 붙은 클래스는 JPA가 관리하며 엔티티라고 한다.
* JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 사용이 필수다.
* final 클래스, enum, 인터페이스, inner 클래스에는 선언할 수 없다.
* JPA는 내부적으로 엔티티 클래스를 리플렉션을 통해 생성하기 때문에 기본 생성자가 필수다.
* 저장할 필드는 final 불변으로 선언할 수 없다.
* name 속성을 통해 JPA에서 사용할 엔티티 이름을 지정할 수 있으나 가급적 클래스 이름을 그대로 사용한다.

<br>

## 2. @Table

* 엔티티와 매핑할 테이블 이름을 별도로 지정한다.
  * 지정하지 않는다면 엔티티 이름과 동일한 테이블을 맵핑한다.

<br>

## 3. DB 스키마 자동 생성

* 예전에 공부한 내용이라 [Spring Data JPA의 DB 초기화 전략](https://xlffm3.github.io/spring%20data/spring-db-initialization/) 글을 참고한다.
  * Hibernate의 경우 XML 파일에 ``<property name="hibernate.hbm2ddl.auto" value="create"/>``와 같이 옵션을 지정해주면 된다.
* 데이터베이스 방언을 활용해서 데이터베이스에 맞는 DDL을 애플리케이션 실행 시점에 자동 생성하여 실행한다.
  * @Entity 클래스들을 스캔하여 그에 맞는 테이블들을 자동으로 생성해준다.
  * 운영 장비에서는 create, create-drop, update 옵션을 절대 사용하지 않는다.

<br>

## 4. DDL 생성 기능

* DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 끼치지 않는다.
  * ``@Column(nullable = false, length = 10)``
    * 유니크 제약을 칼럼 레벨에서 줄 수 있으나, 제약 이름이 랜덤한 문자열이 되어 가독성이 나쁘다.
  * ``@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE",  columnNames = {"NAME", "AGE"} )})``

<br>

## 5. 맵핑 애너테이션

> Member.java

```java
@Entity
public class Member {

    @Enumerated(EnumType.STRING)
    private Role role;
    private LocalDateTime createTime;
    @Id
    private Long id;
    @Column(name = "name", nullable = false, length = 20)
    private String userName;
    @Transient
    private Long except;
}
```

> SQL

```sql
create table Member (
   id bigint not null,
    createTime timestamp,
    role varchar(255),
    name varchar(20) not null,
    primary key (id)
)
```

* @Column
  * 필드와 맵핑할 칼럼의 실제 이름이나 길이 및 제약 조건(nullable, 유일키) 등을 지정할 수 있다.
* @Temporal
  * DB는 Date, Time, TimeStamp 등 날짜와 시간의 조합에 따른 타입이 다양하다.
    * Java Date나 Calendar 타입을 사용하면 속성으로 ``TemporalType.DATE``, ``TemporalType.TIME``, ``TemporalType.TIMESTAMP`` 등을 지정해준다.
  * LocalDate나 LocalDateTime을 사용하면 해당 애너테이션이 필요없다!
* @Enumerated : Enum 타입을 맵핑할 때 사용한다.
  * 속성은 항상 ``EnumType.STRING``를 사용함으로써 Enum의 String값을 통해 저장 및 조회되도록 한다.
  * Effective Java에 나온 것 처럼 ``EnumType.ORDINAL``을 사용하면 변경에 취약하다.
* @Lob : DB의 CLOB(문자열) 혹은 BLOB(그 외) 타입과 맵핑된다.
* @Transient : 특정 필드를 칼럼에 맵핑하지 않으며, DB에서 저장 및 조회되지 않는다.

<br>

## 6. 기본 키 맵핑 방법

* 직접 할당할 때는 @ID만 사용하고, 자동 생성할 때는 @GeneratedValue를 함께 붙인다.
* 전략 속성.
  * IDENTITY : 데이터베이스에 위임하며 MYSQL에서 주로 사용한다.
  * SEQUENCE : 데이터베이스 시퀀스 오브젝트를 사용하며 ORACLE에서 주로 사용한다.
    * @SequenceGenerator가 필요하다.
  * TABLE : 키 생성용 테이블을 사용하며, 모든 DB에서 사용할 수 있다.
    * @TableGenerator가 필요하다.
  * AUTO : 방언에 따라 자동 지정하며 기본값이다.

### 6.1. IDENTITY

* 기본 키 생성을 데이터베이스에 위임한다.
* 주로 MySQL, PostgreSQL, SQL Server, DB2 등에서 사용 한다.
  * MySQL은 AUTO_INCREMENT다.

> Member.java

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

> Application.java

```java
Member member = new Member();

System.out.println("========");
entityManager.persist(member);
System.out.println("========");
entityManager.find(Member.class, 1L);

transaction.commit();
```

* JPA는 쓰기 지연을 통해 INSERT 쿼리를 트랜잭션 커밋 시점에 날린다.
* ``persist()``를 호출할 때 엔티티 인스턴스가 영속성 컨텍스트의 1차 캐시에 저장되어야 한다.
  * 1차 캐시에는 @ID(PK)와 엔티티 정보 등이 담긴다.
  * 문제는 현재 기본키 생성을 DB에 위임한 상태라서 데이터베이스에 INSERT SQL을 실행하기 전에는 ID PK를 알 수 없다.
* 따라서 IDENTITY 전략은 ``persist()`` 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회한다.
  * PK를 알게되면 엔티티 정보가 1차 캐시에 저장되기 때문에 ``find()``를 호출할 때는 별도의 조회 쿼리를 날리지 않게 된다.

### 6.2. SEQUENCE

* 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다.
* 오라클, PostgreSQL, DB2, H2 데이터베이스 등에서 사용된다.

> Member.java

```java
@Entity
@SequenceGenerator(name = "sequence-generator", sequenceName = "MEMBER_SEQ", initialValue = 1, allocationSize = 1)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "sequence-generator")
    private Long id;
}
```

> SQL

```sql
drop sequence if exists MEMBER_SEQ
Hibernate: create sequence MEMBER_SEQ start with 1 increment by 1
```

* SEQUENCE를 사용하며 눈 여겨볼 점은 ``allocationSize`` 속성이다.
* IDENTITY는 ``persist()`` 호출 시점에 엔티티의 PK를 알지 못해 영속성 컨텍스트에 엔티티를 저장하지 못하므로, 호출 시점에 INSERT 쿼리를 바로 날린다.
* 반면 SEQUENCE는 ``persist()`` 호출 시점에 ``call next value for MEMBER_SEQ`` 호출하여 PK를 얻어온다.
  * 결과적으로 한 번 쓰기 작업을 수행할 때 INSERT 쿼리를 포함해 두 번의 쿼리를 날리는 문제가 발생한다.
* ``allocationSize``의 값은 시퀀스 한 번 호출에 증가하는 수를 의미한다.
  * 기본 값은 50이며, 한 번 호출하면 DB에 저장된 시퀀스의 현재 값이 50만큼 증가한다.
  * 시퀀스 초기 값이 1이면 1~50에 해당하는 시퀀스가 어플리케이션의 메모리에 저장된다.
  * 어플리케이션에서 ``persist()``를 호출하여 엔티티를 영속시킬 때마다 DB 시퀀스에게 쿼리를 날려 PK를 얻지 않고, 메모리에 적재된 시퀀스를 통해 순서대로 PK를 얻는다.
    * PK를 얻기 위한 불필요한 조회 쿼리를 생략하게 된다.
  * 즉, 시퀀스에게 PK를 알려달라고 요청할 때 값을 하나가 아닌 여러 개를 미리 받아오는 것이다.
  * 계속된 쓰기 작업을 통해 미리 가져온 PK 시퀀스를 50까지 다 쓰면, 그제서야 ``call next value for MEMBER_SEQ``를 날려 다음 값을 요청한다.
    * 그 다음에는 51~100에 해당하는 시퀀스 값들이 반환될 것이다.
  * 최적화를 위해 사용한다.

### 6.3. TABLE

> Member.java

```java
@Entity
@TableGenerator(name = "table-generator", table = "MY_SEQUENCES", pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "table-generator")
    private Long id;
}
```

> SQL

```java
create table MY_SEQUENCES (
   sequence_name varchar(255) not null,
    next_val bigint,
    primary key (sequence_name)
);
```

* 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다.
* 모든 DB에서 적용이 가능하지만 성능 단점이 존재한다.
* 영속성 컨텍스트가 PK를 알아야할 때마다 시퀀스 테이블로부터 PK를 조회하고, next_val 컬럼을 +1하는 UPDATE 쿼리를 보내서 값을 증가시킨다.
  * 1번의 쓰기 작업을 하는데 2번의 쿼리가 추가로 발생한다.
  * TABLE 전략 또한 ``allocationSize`` 속성을 통해 어느정도 최적화가 가능하다.

### 6.4. 권장 전략

* 기본 키 제약 조건?
  * not null, 유일, 변하면 안 된다.
* 미래까지 **변하지 않는다** 조건을 만족하는 자연키를 찾기 어렵다.
  * 주민등록번호 또한 기본 키로 적합하지 않다.
* 따라서 대리키(대체키)를 사용해야 한다.
* 권장하는 방식?
  * Long형 + 대체키 + 키 생성전략 형태를 사용한다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
