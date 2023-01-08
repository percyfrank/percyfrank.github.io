---
title: "[Java ORM 표준 JPA 프로그래밍] 11장 : 객체지향 쿼리 언어 - 중급 문법"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-07-01
last_modified_at: 2021-07-01
---

## 1. 경로 표현식

* .(점)을 찍어 객체 그래프를 탐색하는 것이다.
* 상태 필드 : 단순히 값을 저장하기 위한 필드 다.
* 연관 필드 : 탐색 대상이 컬렉션 혹은 엔티티 등 연관 관계를 위한 필드다.

### 1.1. 특징

* 상태 필드는 경로 탐색의 끝이며 더이상 탐색이 불가능하다.
* 단일 값 연관 경로: 묵시적 내부 조인(inner join)이 발생하며, 추가적인 탐색이 가능하다.
* 컬렉션 값 연관 경로 : 묵시적 내부 조인이 발생하며, 추가적인 탐색이 불가능하다.
  * 단, FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 추가적인 탐색이 가능하다.
  * ``select t.members.username from Team t``는 실패하지만 ``select m.username from Team t join t.members m``는 성공한다.
* 명시적 조인 : ``select m from Member m join m.team t``
* 묵시적 조인 : 경로 표현식에 의해 묵시적으로 생성되는 SQL을 의미하며, 내부 조인만 가능하다.

### 1.2. 예제

> 비교

```sql
[JPQL] 
select o.member from Orders o

[SQL] 
select m.* from Orders o 
inner join Member m on o.member_id = m.id
```

### 1.3. 주의사항

* 가급적 묵시적 조인 대신에 명시적 조인을 사용한다.
* 조인은 SQL 튜닝에 중요하지만, 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.
  * 운영 환경에서 묵시적 조인을 유발하는 JPQL 쿼리를 사용하면 엔티티를 조회하는데 수 많은 내부 조인이 발생해 성능 하락으로 이어진다.

<br>

## 2. Fetch Join

![image](https://user-images.githubusercontent.com/56240505/124123894-c2abb980-dab2-11eb-9ba4-b76a13723d05.png)

> Application.java

```java
List<Member> resultList = entityManager.createQuery("select m from Member m", Member.class)
        .getResultList();
System.out.println("==========");
for (Member m : resultList) {
    System.out.println("iter " + m.getName() + " ___ " + m.getTeam());
}
```

> SQL

```sql
Hibernate:
    /* select
        m
    from
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.name as name3_0_,
            member0_.team_id as team_id4_0_
        from
            Member member0_
==========
Hibernate:
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_
    from
        Team team0_
    where
        team0_.id=?
iter a ___ Ta
Hibernate:
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_
    from
        Team team0_
    where
        team0_.id=?
iter b ___ Tb
```

* 지연 로딩으로 인해 반복문을 돌 때마다 Team에 대한 조회 쿼리를 날린다.
  * 대표적인 N + 1 문제다.
* 즉시 로딩으로 설정을 변경해도 JPQL을 사용하면 시점의 차이일 뿐, N + 1 문제는 해결할 수 없다.
  * JPA의 기본 전략은 조인을 사용하기 때문에 EntityManager로 Member 엔티티를 조회할 때는 N + 1 문제가 생기지 않는다.
  * 그러나 JPQL은 단순히 SQL로 쿼리를 변환시키기 때문에 JPQL을 사용하면 지연 로딩의 예제처럼 N + 1 문제가 발생한다.

> 비교

```sql
[JPQL] 
select m from Member m join fetch m.team

[SQL] 
SELECT M.*, T.* FROM MEMBER M 
INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

* SQL 조인 종류가 아니며, JPQL에서 성능 최적화를 위해 제공하는 기능이다.
* 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능이며, ``join fetch`` 명령어를 사용한다.

### 2.1. Collection Fetch Join

> 비교

```sql
[JPQL] 
select t  from Team t join fetch t.members 

[SQL] 
SELECT T.*, M.*  FROM TEAM T 
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID 
```

![image](https://user-images.githubusercontent.com/56240505/124126151-55e5ee80-dab5-11eb-9922-500f1fe80a5d.png)

* 일대다 관계, 컬렉션 페치 조인의 경우다.

> Application.java

```java
Ta members : [Member{name='a'}, Member{name='b'}]
Ta members : [Member{name='a'}, Member{name='b'}]
Tb members : [Member{name='c'}]
```

* 일대다 관계에서 페치 조인을 사용하면 중복된 결과가 나올 수 있다.
* SQL의 DISTINCT는 중복된 결과를 제거하는 명령이며, JPQL의 DISTINCT는 2가지 기능을 제공한다.
  * SQL에 DISTINCT를 추가한다.
  * 애플리케이션에서 엔티티 중복을 제거한다.

> Application.java

```java
List<Team> resultList = entityManager.createQuery("select distinct t from Team t join fetch t.members", Team.class)
        .getResultList();
```

![image](https://user-images.githubusercontent.com/56240505/124127079-44e9ad00-dab6-11eb-9a1a-7d99145c5c04.png)

* SQL은 DISTINCT를 추가하더라도, 모든 칼럼의 데이터가 동일하지 않으면 동일한 데이터로 인식하지 않아 중복 제거에 실패한다.

![image](https://user-images.githubusercontent.com/56240505/124127153-5cc13100-dab6-11eb-8c87-bc5c31fc20e2.png)

* 어플리케이션(영속성 컨텍스트)에서 추가적으로 같은 식별자에 대한 엔티티를 제거하려고 시도한다.

### 2.2. 차이점 정리

* JPQL의 일반 조인은 실행시 연관된 엔티티를 함께 조회하지 않는다.
  * JPQL은 결과를 반환할 때 연관 관계를 고려하지 않고, SELECT 절에 지정한 엔티티만 조회한다.
* 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)된다.
  * 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다.

### 2.3. 주의점

> Application.java

```java
List<Team> resultList = entityManager.createQuery("select t from Team t join fetch t.members as m where m.age >= 19", Team.class)
        .getResultList();
```

* 페치 조인 대상에는 별칭을 줄 수 없으며, 하이버네이트는 가능하지만 사용을 가급적 자제한다.
  * JPA는 엔티티의 객체 그래프를 완벽하게 조회할 수 있어야 하지만, 페치 조인에서 별칭을 쓰게 되면 연관 객체들이 누락되버릴 수 있다.
  * 어떤 경우에는 팀과 관련된 멤버들이 완벽하게 존재하고, 어떤 경우는 아니라면 데이터 정합성 이슈가 발생할 수 있다.
* 둘 이상의 컬렉션은 페치 조인할 수 없다.
  * Team-Member와 같은 일대다 관계에서도 페치 조인을 사용하면 중복되는 데이터들이 많이 발생한다.
  * 둘 이상의 컬렉션을 페치 조인하면 곱하게 되는 데이터의 양이 너무 많이 발생할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/124257490-6060c080-db67-11eb-883b-b6c0340dd5db.png)

> Application.java

```java
List<Team> teamFromJpql = entityManager.createQuery("select t from Team t join fetch t.members", Team.class)
        .setFirstResult(0)
        .setMaxResults(1)
        .getResultList();
```

> SQL & LOG

```sql
Hibernate:
    /* select
        t
    from
        Team t
    join
        fetch t.members */ select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.name as name3_0_1_,
            members1_.team_id as team_id4_0_1_,
            members1_.team_id as team_id4_0_0__,
            members1_.id as id1_0_0__
        from
            Team team0_
        inner join
            Member members1_
                on team0_.id=members1_.team_id

WARN: HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```

* 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
  * 일대일, 다대일 같은 단일값 연관 필드들은 페치 조인해도 페이징이 가능하다.
* 컬렉션을 페치 조인하면 하이버네이트는 경고 로그를 남기고, 메모리에서 페이징하며 매우 위험하다.
  * 일반적인 페이징은 ROWNUM, OFFSET, LIMIT을 활용하지만, 컬렉션 페치 조인시 모든 값들을 가져온다.
  * 일대다와 같은 컬렉션 페치 조인의 경우 사진과 같이 동일한 PK의 엔티티에 대해 여러 값들이 맵핑되기 쉽다.
    * JPA는 DB가 반환하는 데이터를 엔티티 객체로 변환시킬 뿐, 어떤 데이터가 반환될지는 모른다.
    * 페이징할 때마다 반환되는 값이 다를 수도 있는 등 정합성 문제가 존재한다.

> Member.java

```java
@BatchSize(size = 100)
@OneToMany(fetch = FetchType.LAZY, mappedBy = "team")
private List<Member> members = new ArrayList<>();
```

> Application.java

```java
List<Team> teamFromJpql = entityManager.createQuery("select t from Team t", Team.class)
        .setFirstResult(0)
        .setMaxResults(2)
        .getResultList();

System.out.println("iteration start");
for (Team t : teamFromJpql) {
    System.out.println(t.getMembers());
}
```

> SQL

```sql
Hibernate:
    /* select
        t
    from
        Team t */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_
        from
            Team team0_ limit ?
iteration start
Hibernate:
    /* load one-to-many practice.docs.spring.domain.Team.members */ select
        members0_.team_id as team_id4_0_1_,
        members0_.id as id1_0_1_,
        members0_.id as id1_0_0_,
        members0_.age as age2_0_0_,
        members0_.name as name3_0_0_,
        members0_.team_id as team_id4_0_0_
    from
        Member members0_
    where
        members0_.team_id in (
            ?, ?
        )
[Member{name='a'}, Member{name='b'}]
[Member{name='c'}]
```

* 지연 로딩 전략을 사용하면 반복문을 순회할 때 마다 매 Team 엔티티와 관련된 Members 엔티티 프록시들에 대해 초기화 작업이 발생한다.
  * 즉, N+1 문제가 발생한다.
* 컬렉션을 사용할 때는 페치 조인을 적용하기 어렵기 때문에, BatchSize를 통해 문제를 해결한다.
  * BatchSize를 적용하면 JPQL로 가져온 Team의 Members를 지연 로딩할 때, 현재 접근한 Team 엔티티뿐만 아니라 리스트에 담긴 다른 Team 아이디들 또한 IN 구로 넘긴다.
  * 연관된 Member 엔티티의 지연 로딩에 대한 N개의 쿼리를 줄여준다.
  * BatchSize는 persistence.xml에 글로벌하게 세팅할 수 있으며, 주로 크기는 1000 이하로 잡는다.

### 2.4. 정리

* 페치 조인은 연관된 엔티티들을 SQL 한 번으로 조회하여 성능을 최적화한다.
* @OneToMany(fetch = FetchType.LAZY)과 같은 글로벌 전략보다 우선된다.
* 실무는 글로벌 로딩 전략을 모두 지연 로딩하며, 최적화가 필요한 곳에 페치 조인을 적용한다.
* 모든 것을 페치 조인으로 해결할 수는 없다.
  * 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다.
* 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 한다면, 페치 조인보다는 일반 조인을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적이다.

<br>

## 3. 다형성 쿼리

> SQL

```sql
[JPQL] 
select i from Item i  where type(i) IN (Book, Movie)

[SQL] 
select i from i  where i.DTYPE in (‘B’, ‘M’)
```

* type은 조회 대상을 특정 자식으로 한정시킨다.

> SQL

```sql
[JPQL] 
select i from Item i  where treat(i as Book).auther = ‘kim’

[SQL] 
select i.* from Item i  where i.DTYPE = ‘B’ and i.auther = ‘kim’
```

* treat은 타입 캐스팅과 유사하며, 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용된다.

<br>

## 4. 엔티티 직접 사용

> Application.java

```java
Team team = em.find(Team.class, 1L);
String qlString = "select m from Member m where m.team = :team";
List resultList = em.createQuery(qlString).setParameter("team", team).getResultList();
//select m.* from Member m where m.team_id=?
```

> Application.java

```java
String jpql = "select m from Member m where m.id = :memberId";
List resultList = em.createQuery(jpql).setParameter("memberId", memberId).getResultList();
//select m.* from Member m where m.id=?
```

* JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본키 값 혹은 외래키 값을 사용한다.
  * 엔티티는 식별자(기본키 혹은 외래키)로 구분할 수 있기에 JPQL에 엔티티를 직접 제공하거나, 식별자 아이디를 제공할 수 있다.
    * 두 방식 모두 같은 SQL이 실행된다.

<br>

## 5. Named 쿼리

> Member.java

```java
@Entity
@NamedQuery(name = "Member.findByUsername",
            query="select m from Member m where m.username = :username")
public class Member {
    ...
}
```

> Application.java

```java
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                            .setParameter("username", "회원1")
                            .getResultList();
```

* 미리 정의해서 이름을 부여해두고 사용하는 JPQL로서 정적 쿼리이다.
* 어노테이션이나 XML에 정의한다.
  * XML이 항상 우선권을 가진다.
  * 어플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다.
* 어플리케이션 로딩 시점에 초기화 후 재사용하며, 해당 시점에 쿼리를 검증한다.
* Spring Data Jpa의 @Query가 내부적으로 @NamedQuery를 사용한다.

<br>

## 6. 벌크 연산

* 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?
* 더티 체킹 기능으로 실행하려면 너무 많은 SQL이 실행된다.
  * 재고 10개 미만인 상품 엔티티 리스트를 조회하고, 모든 엔티티들의 가격을 변경시킨다.
  * 변경된 데이터가 100건이라면 100번의 UPDATE SQL이 실행된다.
* 벌크 연산은 쿼리 한 번으로 여러 테이블 로우(엔티티)를 변경한다.
* ``executeUpdate()`` 리턴 값은 영향받은 엔티티의 수이며, 쓰기 작업 모두를 지원한다.

> Application.java

```java
Member member = new Member(13);
entityManager.persist(member);
int rowCounts = entityManager.createQuery("update Member m set m.age = 20")
        .executeUpdate();
//entityManager.clear();
Member findMember = entityManager.find(Member.class, member.getId());
System.out.println(findMember.getAge()); //13
```

* 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리하기 때문에 주의한다.
  * 다른 연산들보다 가장 먼저 벌크 연산을 수행한다.
  * 혹은 벌크 연산 수행 직후 영속성 컨텍스트를 초기화해준다.
* JPQL이 실행될 때 플러시되어도 영속성 컨텍스트는 초기화되지 않는다.
  * 위의 경우 조회한 Member가 현재 영속성 컨텍스트의 1차 캐시에 존재하는 Member다.
  * 나이가 20으로 변경 반영이 되지 않고 여전히 13으로 나온다.
  * 따라서 데이터 정합성을 위해 벌크 연산을 가장 먼저 수행하거나, 수행 직후 영속성 컨텍스트를 초기화해야 한다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
