---
title: "[Java ORM 표준 JPA 프로그래밍] 10장 : 객체지향 쿼리 언어 - 기본 문법"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-29
last_modified_at: 2021-06-29
---

## 1. JPA가 지원하는 쿼리 방법들

* JPQL
* JPA Criteria
* QueryDSL
* Native SQL
* 기타
  * JdbcAPI 직접 사용하기.
  * MyBatis 및 Spring JdbcTemplate 함께 사용하기.

<br>

## 2. JPQL

* JPA를 사용하면 엔티티 객체를 중심으로 개발하지만, 문제는 검색 쿼리다.
* 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색하지만, 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
* 어플리케이션이 필요한 데이터만 DB에서 조회하려면, 검색 조건이 포함된 SQL이 필요하다.
* JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.
  * SQL과 문법이 유사하다.
* JPQL은 엔티티 객체를 대상으로 검색하는 객체 지향 쿼리(SQL)이며, SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
  * SQL을 추상화하기에 특정 데이터베이스 SQL에 의존하지 않는다.
  * **결국 각 DB 방언에 맞는 적합한 SQL로 변환된다.**

### 2.1. 기본 예제

> Application.java

```java
String sql = "select m from Member m where m.age > 18";
List<Member> resultList = entityManager.createQuery(sql, Member.class)
        .getResultList();
```

> SQL

```sql
Hibernate:
    /* select
        m
    from
        Member m
    where
        m.age > 18 */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.team_id as team_id4_0_
        from
            Member member0_
        where
            member0_.age>18
```

<br>

## 3. Crieria와 QueryDSL

> Application.java

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;
List<Member> list = query.selectFrom(m)
        .where(m.age.gt(18))
        .orderBy(m.name.desc())
        .fetch();
```

* Crieria는 문자가 아닌 Java 코드로 JPQL을 작성하는 등, 빌더 기능을 제공한다.
* JPA 공식 기능이지만 너무 복잡하고 실용성이 없기에 QueryDSL 사용을 권장한다.
  * QueryDSL 또한 JPQL 빌더 역할을 하며, 컴파일 시점에 문법 오류를 찾을 수 있다.
  * 동적 쿼리 작성이 편리하며 단순하고 쉽다.

<br>

## 4. Native Query

> Application.java

```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = kim";
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```

* JPA가 제공하는 SQL을 직접 사용하는 기능으로, JPQL로 해결할 수 없는 특정 DB에 의존적인 기능을 수행할 때 쓴다.
  * 오라클의 Connected By와 같이 특정 DB만 사용하는 힌트 등.

<br>

## 5. JDBC API 직접 사용 등 유의사항

* JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, JdbcTemplate 및 마이바티스등을 함께 사용이 가능하다.
* 단, JPQL과 다르게 자동으로 플러시가 되지 않으니 영속성 컨텍스트를 적절한 시점에 강제로 플러시해야 한다.
  * JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시한다.

<br>

## 6. JPQL 기본 문법

> Application.java

```java
Query query = entityManager.createQuery("SELECT m FROM Member m where m.name=:username");
query.setParameter("username", "a");
Member findMember = (Member) query.getSingleResult();
assert findMember.getAge() == 19;
```

* 엔티티와 속성은 대소문자를 구분한다.
* SELECT와 FROM 등 JPQL 예약어 키워드는 대소문자를 구분하지 않는다.
* 엔티티 이름을 사용할 뿐, 테이블 이름이 아니다.
* 별칭은 필수지만 as는 생략이 가능하다.
* 집계 함수와 GROUP BY, HAVING 모두 제공한다.
  * ``sum(m.age)``, ``max(m.age)`` 등.
* 반환 타입이 명확할 때는 타입을 제공하여 제네릭스를 사용하는 TypedQuery를, 명확하지 않으면 Query를 사용한다.
* 결과 조회 API는 JdbcTemplate처럼 복수 조회(리스트)하는 경우 문제가 없으나, 단일 조회일 때는 예외에 유념한다.
  * 결과가 없거나 2개 이상이면 예외가 발생한다.
* 쿼리에 파라미터를 바인딩할 때 JdbcTemplate처럼 위치를 기반으로 바인딩하거나, NamedJdbcTemplate처럼 파라미터 이름을 기반으로 바인딩한다.

<br>

## 7. 프로젝션

* 프로젝션이란 SELECT 절에 조회할 대상을 지정하는 것을 의미한다.
  * 대상 : 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자열 등 기본 데이터 타입) 등.
* ``SELECT m FROM Member m`` : 엔티티 프로젝션
  * 결과로 조회되는 Member 엔티티들은 모두 영속성 컨텍스트에서 관리된다.
* ``SELECT m.team FROM Member m`` : 엔티티 프로젝션
  * ``SELECT t from Member m join m.team t``로 표현하는 것이 더 바람직하다.
* ``SELECT m.address FROM Member m`` : 임베디드 타입 프로젝션
  * Address는 @Embeddable 클래스다.
* ``SELECT m.username, m.age FROM Member m`` : 스칼라 타입 프로젝션
* DISTINCT로 중복 제거한다.

### 7.1. 스칼라 타입 조회 방법

* ``SELECT m.username, m.age FROM Member m`` 조회를 어떻게 할까?
  * Query 타입으로 조회한다.
  * Object[] 타입으로 조회한다.
  * new 명령어로 조회한다. 
    * ``SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m``
    * 패키지명을 포함한 전체 클래스명을 입력해야 한다.
    * 순서와 타입이 일치하는 생성자가 필요하다.

<br>

## 8. 페이징 API

> Application.java

```java
List<Member> resultList = entityManager.createQuery("SELECT m from Member m order by m.age desc", Member.class)
        .setFirstResult(1)
        .setMaxResults(5)
        .getResultList();
```

> SQL

```sql
Hibernate:
    /* SELECT
        m
    from
        Member m
    order by
        m.age desc */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.team_id as team_id4_0_
        from
            Member member0_
        order by
            member0_.age desc limit ? offset ?
```

* 간편하게 페이징을 지원해준다.
  * 각 DB 방언에 맞는 SQL이 생성된다.

<br>

## 9. Join

> Application.java

```java
entityManager.createQuery("SELECT m from Member m inner join m.team t", Member.class);
entityManager.createQuery("SELECT m from Member m left join m.team t", Member.class);
```

* 일반적인 Join은 기존 SQL과 크게 다를 것이 없다.

> Application.java

```java
entityManager.createQuery("SELECT m from Member m, Team t where m.name = t.name", Member.class);
```

* 세타 조인 쿼리이며, 실행 결과 크로스 조인이 발생한다.

> Application.java

```java
entityManager.createQuery("SELECT m from Member m left join m.team t on m.name = t.name", Member.class);
```

* 조인 대상 필터링이 가능하며, 연관 관계 없는 엔티티의 외부 조인도 가능하다.

<br>

## 10. 서브 쿼리

* ``select m from Member m  where m.age > (select avg(m2.age) from Member m2)``
* ``select m from Member m  where (select count(o) from Order o where m = o.member) > 0``
* 지원 함수 : 크게 어려운 내용은 없다.
  * [NOT] EXISTS (subquery)
    * {ALL | ANY | SOME} (subquery)
  * [NOT] IN (subquery)
  * ``select m from Member m  where exists (select t from m.team t where t.name = ‘팀A')``
  * ``select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)``

### 10.1. 한계

* JPA는 WHERE, HAVING 절에서만 서브 쿼리가 사용이 가능하다.
  * 하이버네이트는 SELECT 절도 지원해준다.
* FROM 절의 서브 쿼리는 현재 JPQL에서 불가능하며, 조인으로 풀 수 있으면 풀어서 해결한다.

<br>

## 11. JPQL 표현식

> Application.java

```java
List<Item> resultList = entityManager.createQuery("SELECT m from Member where m.type = jpabook.MemberType.Admin and type(m) = Album", Item.class)
        .getResultList();
```

* EXISTS, IS NULL, COALESCE, NULLIF 등은 일반 SQL과 표현식이 같다.
* 문자 : ‘HELLO’, ‘She’’s’
* Boolean: TRUE, FALSE
* 숫자: 10L(Long), 10D(Double), 10F(Float)
* ENUM: jpabook.MemberType.Admin
  * 패키지 명을 포함해야 한다.
* 엔티티 타입 : ``TYPE(m) = Member``
  * 상속 관계에서 사용한다.
  * DTYPE을 통해 식별한다.

### 11.1. 표준 함수 및 사용자 정의 함수

> MyH2Dialect.java

```java
public class MyH2Dialect extends H2Dialect {

    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }
}
```

> persistence.xml

```xml
<property name="hibernate.dialect" value="practice.docs.spring.domain.MyH2Dialect"/>
```

> Application.java

```java
String query = "select function('group_concat', i.name) from Item i";
String query2 = "select group_concat(i.name) from Item i";
```

* JPQL에서 제공하는 표준 함수들은 필요할 때마다 찾아본다.
* 사용자 정의 함수를 정의해서 사용할 수 있다.
  * 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다.

<br>
---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
