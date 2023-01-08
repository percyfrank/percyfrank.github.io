---
title: "[Java ORM 표준 JPA 프로그래밍] 1장 : JPA 소개"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-18
last_modified_at: 2021-06-18
---

## 1. SQL 중심적인 개발의 문제

* 패러다임의 불일치가 발생한다.
  * OOP는 속성과 기능을 묶어 캡슐화하며, 추상화와 상속 및 다형성 등을 통해 시스템의 복잡성을 제어하는 기능을 제공한다.
  * 관계형 DB는 데이터를 정규화해서 저장하는 것이 목적이다.
* 객체를 관계형 DB에 저장 및 관리하기 위해서는?
  * 객체와 SQL간의 변환 작업이 요구되며, 개발자가 SQL 맵퍼의 역할을 수행한다.
  * 단순 반복적인 CRDU 및 맵핑 코드가 반복되며, 엔티티에 필드가 추가되면 해당 엔티티를 사용 중인 SQL을 많이 수정해야 한다.

<br>

## 2. OOP와 RDB의 차이

### 2.1. 상속

* 객체의 상속을 RDB 상에서 슈퍼타입과 서브타입 테이블간의 관계로 표현할 수 있다.
* 데이터를 저장할 때 슈퍼타입 테이블과 서브타입 테이블 각각에 Insert 쿼리를 날려야 한다.
* 조회할 때에도 여러 테이블에 대해 Join SQL을 작성해야 하며, 자식 타입에 맞는 각각의 객체를 생성해야 한다.
  * 매우 번거로워서 DB에 저장할 객체에는 상속을 잘 안 쓴다.

### 2.2. 연관관계

* 객체는 참조를 통해 다른 객체들과 연관관계를 맺는다.
* 반면 테이블은 외래키를 통해 다른 테이블을 참조함으로써 연관관계를 맺는다.

> Member.java

```java
class Member {
    String id; //MEMBER_ID 컬럼 사용
    Long teamId; //TEAM_ID FK 컬럼 사용 //**
    String username;//USERNAME 컬럼 사용
}
class Team {
    Long id; //TEAM_ID PK 사용
    String name; //NAME 컬럼 사용
}
```

* 객체를 테이블에 맞춰 모델링한다면 SQL 쿼리 작성은 쉽겠지만 객체지향적인 모델링이 아니다.
  * Member가 Team을 조회할 때 단순히 getter를 사용하지 못한다.
* 그러나 객체지향적으로 모델링하면 SQL을 작성할 때 번거롭다.
  * Member 객체를 테이블에 저장할 때, Member가 가진 TeamId를 얻으려면 ``member.getTeam().getId()`` 형태가 되어야 한다.

> MemberDao.java

```java
/*
SELECT M.*, T.* FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
*/
public Member find(String memberId) {
    //SQL 실행 ...
    Member member = new Member();
    //데이터베이스에서 조회한 회원 관련 정보를 모두 입력
    Team team = new Team();
    //데이터베이스에서 조회한 팀 관련 정보를 모두 입력
    //회원과 팀 관계 설정
    member.setTeam(team); //**
    return member;
}
```

* 엔티티를 조회할 때에도 여러 테이블간의 Join이 수반된다.
  * 모은 데이터들로 엔티티를 생성하고, setter를 통해 관계를 설정해준다.
* SQL에 따라서 객체 그래프 탐색 범위가 결정된다.
  * 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.
  * SQL이 Order 테이블에 대해 Join하지 않으면 ``member.getOrder()``는 null이 반환되며, 이는 사용하는 클라이언트 측에서 엔티티에 대한 신뢰 문제가 발생한다.
* 레이어드 아키텍쳐는 기본적으로 인접한 레이어를 신뢰할 수 있어야하지만, 이로 인해 진정한 계층 분할이 어렵다.

### 2.3. 데이터 식별 방법

* DAO가 동일 식별자로 조회한 두 엔티티 객체는 new 연산자를 사용하기 때문에 레퍼런스가 다른 객체로 인식한다.

### 2.4. 시사점

* 객체답게 모델링할수록 맵핑 작업만 늘어난다!
* 객체를 SQL에 맞춰서 데이터를 전송하는 역할로만 사용한다.
* 객체를 Java 컬렉션에 저장하듯이 DB에 저장할 수 없을까?
  * ``add()``, ``get()``으로 쉽게 객체를 조회 및 저장할 수 있다.
  * 레퍼런스 참조이기 때문에 가져온 원소를 setter로 수정하면 컬렉션에 존재하는 원소에 수정이 자동 반영된다.
  * ``Parent parent = list.getChild(0);``와 같이 컬렉션은 다형성을 사용하기 쉽다.

<br>

## 3. JPA(Java Persistence API)

![image](https://user-images.githubusercontent.com/56240505/122545881-515b1800-d069-11eb-9493-b55182d8849d.png)

* ORM(Object-Relational Mapping) : 객체를 RDB와 맵핑하는 기술이다.
  * 객체는 객체대로 설계한다.
  * RDB는 RDB대로 설계한다.
  * ORM 프레임워크가 중간에서 이 둘을 맵핑해준다.
* 엔티티를 저장하거나 조회할 때 연관 관계를 분석함으로써 자동으로 SQL을 생성해주고 JDBC API를 호출한다.
  * 패러다임 불일치를 해소한다.
* ORM을 제공하는 EJB의 Entity Bean이 불편한 점이 많아 Hibernate가 개발되었다.
* JPA는 Java 진영의 표준 ORM API 인터페이스 명세다.
  * 구현체는 Hibernate, EclipseLink, DataNucleus가 있다.

### 3.1. 장점

* SQL 중심적인 개발에서 객체 중심으로 개발로 이동하면서 생산성이 높아진다.
* 유지보수하기 편하다.
  * 필드 변경시 추가만 하면 되며, JPA가 SQL 쿼리를 수정해준다.

> AlbumDao.java

```java
jpa.persist(album);
Album album = jpa.find(Album.class, albumId);
```

* 패러다임의 불일치를 해결한다.
  * 메서드만 호출하면 그에 수반되는 복잡한 SQL은 JPA가 알아서 처리해준다.
  * 자유로운 객체 그래프 탐색이 가능하여 반환되는 엔티티를 신뢰할 수 있다.
* 성능이 좋다.
* 데이터 접근 추상화와 벤더 독립성을 표준으로 지원한다.

<br>

## 4. 성능 최적화

> MemberService.java

```java
String memberId = "100";
Member m1 = jpa.find(Member.class, memberId); //SQL
Member m2 = jpa.find(Member.class, memberId); //캐시
println(m1 == m2) //true
```

* 1차 캐시를 통해 동일성을 보장한다.
  * 같은 트랜잭션 안에서는 같은 엔티티를 반환하기에 쿼리를 두 번 날리지 않아 조회 성능이 향상된다.
  * DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장한다.

> MemberService.java

```java
transaction.begin();

em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
transaction.commit();
```

* 트랜잭션을 지원하는 쓰기 지연을 제공한다.
* 트랜잭션을 커밋할 때까지 Insert나 Update SQL을 모은다.
* JDBC Batch SQL 기능을 사용해 한 번에 SQL을 전송한다.
  * batch 코드를 개발자가 직접 짜는 것은 매우 번거로운 과정이지만 이를 편리하게 해준다.

### 4.1. 지연 로딩과 즉시 로딩

> MemberService.java

```java
Member member = memberDAO.find(memberId);
Team team = member.getTeam();
String teamName = team.getName();
```

* 지연 로딩 : 객체가 실제 사용될 때 로딩된다.
  * ``memberDao.find()`` 시점에는 ``SELECT * FROM MEMBER`` 쿼리만 날려 연관 관계 설정이 완성되지 않은 Member 객체를 로드한다.
  * ``team.getName()`` 시점에 ``SELECT * FROm TEAM`` 쿼리를 날려 Team 객체를 로드한다.
    * AOP Proxy 기법 등을 활용한 것이다.
* 즉시 로딩 : JOIN SQL로 한 번에 연관된 객체까지 미리 조회한다.
  * ``memberDao.find()`` 시점에 ``SELECT M.*, T.* FROM MEMBER JOIN ...``와 같은 조인 쿼리를 날려 처음부터 연관 관계가 미리 완벽하게 설정된 엔티티를 반환한다.
* 로직 특성상 엔티티가 처음부터 완벽하게 로드되어야 하는지 혹은 일부만 로드하는게 성능상 유리한지 등을 따져 설정할 수 있다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
