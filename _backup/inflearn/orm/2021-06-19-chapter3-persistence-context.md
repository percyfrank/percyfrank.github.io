---
title: "[Java ORM 표준 JPA 프로그래밍] 3장 : 영속성 관리"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-19
last_modified_at: 2021-06-19
---

## 1. 엔티티 매니저 팩토리와 엔티티 매니저

![image](https://user-images.githubusercontent.com/56240505/122585667-b37e4200-d096-11eb-9bf7-36c5629b6f67.png)

<br>

## 2. 영속성 컨텍스트

* 엔티티를 영구 저장하는 환경을 지칭한다.
* 영속성 컨텍스트란 논리적인 개념으로서 눈에 보이지는 않는다.
  * 엔티티 매니저를 통해 영속성 컨텍스트에 접근한다.
  * 엔티티 매니저 팩토리에서 엔티티 매니저를 생성할 때, 각 엔티티 매니저마다 내부에 영속성 컨텍스트라는 공간이 생긴다고 이해하자.
* 현재 예제와 같은 J2SE 환경에서는 엔티티 매니저와 영속성 컨텍스트가 1:1로 맵핑된다.
* 반면 Spring 등 J2EE 환경에서는 다수의 엔티티 매니저가 영속성 컨텍스트에 N:1로 맵핑된다.
* ``persist()`` 메서드는 DB에 저장하는 것이 아니라 영속성 컨텍스트에 저장하는 것이다.
  * 실제로 트랜잭션이 커밋되는 시점에 DB에 값이 반영된다.

### 2.1. 엔티티 생명 주기

* 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 상태이다.
  * 아직 ``persist()``를 호출하기 전의 엔티티 객체들이 해당된다.
* 영속(managed) : 영속성 컨텍스트에 관리되는 상태이다.
  * ``persist()``나 ``find()``를 통해 영속성 컨텍스트의 1차 캐시에 올라간(저장된) 객체들이 해당된다.
* 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태이다.
  * ``detach()``를 통해 단순히 영속성 컨텍스트에서 분리한 것일 뿐, DB에도 삭제해달라 요청하는 것은 아니다.
* 삭제(removed) : 삭제된 상태이다.
  * ``remove()``를 통해 DB에 실제 DELETE 쿼리를 요청하는 것을 의미한다.

### 2.2. 이점

* 1차 캐시를 제공하며, 동일성(identity)을 보장한다.
* 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)을 제공한다.
  * 위 두 이점은 성능 최적화의 여지를 제공한다.
* 변경 감지(Dirty Checking).
* 지연 로딩(Lazy Loading).

<br>

## 3. 1차 캐시

> Application.java

```java
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
//1차 캐시에 저장됨
em.persist(member);
//1차 캐시에서 조회
Member findMember = em.find(Member.class, "member1");
```

* 엔티티를 ``persist()``로 영속시킬 때, 영속 컨텍스트 내부에 있는 1차 캐시에 해당 엔티티를 저장한다.
* member1 PK에 해당하는 엔티티가 1차 캐시에 존재하기 때문에, 같은 트랜잭션에서 해당 엔티티를 조회할 때 DB까지 가지 않고 캐시에서 가져온다.
  * 실제로 조회 쿼리가 언제 출력되는지 확인해보면 된다.

> Application.java

```java
Member findMember2 = em.find(Member.class, "member2");
```

* 1차 캐시에 없는 엔티티를 조회할 때는 DB에서 조회하고 1차 캐시에 저장한 다음 반환한다.

> Application.java

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");
System.out.println(a == b); //동일성 비교 true
```

* 같은 트랜잭션 내에서 같은 영속성 컨텍스트를 사용하고 있기에 엔티티 동일성이 보장된다.
* 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.

<br>

## 4. 쓰기 지연

> Application.java

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin();
em.persist(memberA);
em.persist(memberB);
//여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
//커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
transaction.commit(); // [트랜잭션] 커밋
```

![image](https://user-images.githubusercontent.com/56240505/122587533-e45f7680-d098-11eb-8982-bd3417b599af.png)

* 영속성 컨텍스트 내부에 쓰기 지연 SQL 저장소가 존재하며, 영속하려는 엔티티를 캐시에 저장한 다음 INSERT SQL을 생성한다.
  * 해당 SQL은 쓰기 지연 SQL 저장소에 저장된다.
* 트랜잭션이 커밋되는 시점에 내부적으로 ``flush()``가 호출되면, 누적된 SQL들이 실제 DB로 넘어가 Batch를 통해 한꺼번에 처리되어 커밋된다.
  * MyBatis 등을 사용할 때 쓰기 작업을 지연해서 커밋하는 시점에 Batch하도록 코드를 구현하는 것은 여간 번거로운 일이 아니다.

> persistence.xml

```xml
<property name="hibernate.jdbc.batch_size" value="10"/>
```

* Batch 사이즈를 설정할 수 있다.

<br>

## 5. 변경 감지

![image](https://user-images.githubusercontent.com/56240505/122588299-dfe78d80-d099-11eb-8df2-72b634c6ea1c.png)

* JPA는 트랜잭션을 커밋하면 내부적으로 ``flush()``를 호출한다.
* 영속성 컨텍스트는 엔티티를 최초로 읽어들인 시점에 스냅샷을 찍어 저장한다.
* 트랜잭션 커밋 시점에 JPA는 캐시에 저장된 엔티티와 스냅샷을 비교하며, 변경된 부분이 있다면 자동으로 UPDATE SQL을 생성하여 반영한다.

> Application.java

```java
EntityManager em = emf.createEntityManager();
EntityTransaction transaction = em.getTransaction();
transaction.begin();

Member memberA = em.find(Member.class, "memberA"); //최초 엔티티 스냅샷 찍은 시점
memberA.setUsername("hi");
memberA.setAge(10);
transaction.commit();
```

* 별도의 ``update()`` 메서드가 없더라도 setter를 통해 값을 변경하면, 트랜잭션 커밋 시점에 더티 체킹을 통해 자동으로 UPDATE 쿼리가 실행된다.
  * 1차 캐시에 저장된 엔티티의 값과 최초 스냅샷의 값이 다르기 때문이다.

<br>

## 6. 플러시

* 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것이다.
* 트랜잭션이 커밋되면 자동으로 ``flush()``가 호출된다.
  * 더티 체킹이 일어난다.
    * 수정된 엔티티에 대한 UPDATE 쿼리가 쓰기 지연 SQL 저장소에 등록된다.
  * INSERT, DELETE, UPDATE 등 쓰기 지연 SQL 저장소의 쿼리들이 DB로 전송되어 Batch 작업을 진행하고 DB에 최종 커밋된다.
* 영속성 컨텍스트를 비우지 않는다.
* 영속성 컨텍스트의 변경 내용을 DB에 동기화한다.
* 트랜잭션이라는 작업 단위가 중요하며, 커밋 직전에만 동기화하면 된다.
  * 이 원칙만 잘 지킨다면, 영속성 컨텍스트와 DB의 데이터 동기화에 대해 크게 신경쓸 필요는 없다.

### 6.1. 방법

* ``EntityManager.flush()``을 직접 호출하기.
* 혹은 트랜잭션을 커밋하면 플러시가 자동으로 호출된다.
* JPQL 쿼리를 실행해도 플러시가 자동으로 호출된다.

> Application.java

```java
Member member = new Member();
member.setId(131L);
member.setName("adfafd");
entityManager.persist(member);
entityManager.flush();

transaction.commit();
```

* 트랜잭션을 커밋하기 전인 ``flush()`` 호출 시점에 INSERT 쿼리가 호출되어 실제로 DB에 변경 내용이 저장된다.
  * ``flush()``를 호출하더라도 영속성 컨텍스트 내용(1차 캐시)가 초기화되거나 하지 않는다.

> Application.java

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

query = em.createQuery("select m from Member m", Member.class);
List<Member> members= query.getResultList();
```

* JPQL 쿼리를 날릴 때 이전의 영속 컨텍스트들이 ``flush()`` 되는 이유?
  * JPQL 쿼리로 Member를 조회하는데 이전에 저장을 목적으로 영속한 memberA, memberB, memberC는 실제 DB에 반영되어 있지 않다.
  * JPQL 쿼리는 SQL 쿼리로 변환되어 실제 DB에 보내지고 데이터를 조회하는데, 조회 결과 memberA, memberB, memberC가 누락된다.
  * 이러한 문제 때문에 JPQL 쿼리를 날리면 자동으로 이전의 영속성 컨텍스트 내용을 전부 ``flush()``한다.
* JPQL 쿼리를 날릴 때 플러시하기 싫으면 ``em.setFlushMode(FlushModeType.COMMIT)``로 커밋할 때만 플러시하게끔 설정할 수 있다.
  * 기본값은 FlushModeType.AUTO다.

<br>

## 7. 준영속 상태

> Application.java

```java
Member memberA = em.find(Member.class, "memberA"); //최초 엔티티 스냅샷 찍은 시점
memberA.setUsername("hi");
memberA.setAge(10);
em.detach(memberA);
transaction.commit();
```


* 영속 상태는 ``persist()`` 혹은 ``find()``를 통해 엔티티가 영속성 컨텍스트의 1차 캐시에 올라간 상태를 의미한다.
  * 즉, 영속성 컨텍스트에 관리되는 상태다.
* 준영속 상태는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)되는 것이다.
* 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.
  * memberA는 영속 상태였으나 ``detach()``를 통해 준영속이 되었기 때문에 더티 체킹이 발생하지 않아 수정 사항이 DB에 반영되지 않는다.

### 7.1. 방법

* ``EntityManager.detach(entity)`` : 특정 엔티티만 준영속 상태로 전환한다.
* ``EntityManager.clear()`` : 영속성 컨텍스트를 완전히 초기화한다.
* ``EntityManager.close()`` : 영속성 컨텍스트를 종료한다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
