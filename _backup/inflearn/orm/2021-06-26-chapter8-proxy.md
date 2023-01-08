---
title: "[Java ORM 표준 JPA 프로그래밍] 8장 : 프록시와 연관관계 관리"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-26
last_modified_at: 2021-06-26
---

## 1. 프록시 기본

* ``find()`` 메서드는 DB에서 실제 엔티티 객체를 조회한다.
* ``getReference()`` 메서드는 DB 조회를 미루는 가짜(프록시) 엔티티 객체를 조회한다.
* 프록시의 특징?
  * 실제 클래스를 상속 받아서 만들어진다.
  * 실제 클래스와 겉 모양이 같다.
  * 사용자는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용한다.
  * 프록시 객체는 내부에 실제 타깃 객체의 레퍼런스를 가지며, 프록시 객체를 호출하면 실제 객체의 메서드를 호출한다.

![image](https://user-images.githubusercontent.com/56240505/123504574-2d18c000-d695-11eb-965b-772fdb3ba20c.png)

* 프록시 객체의 메서드 호출시 실제 타깃 객체가 초기화되지 않았다면, 이 시점에 프록시 객체가 가진 실제 타깃 객체가 초기화된다.

> Application.java

```java
entityManager.flush();
entityManager.clear();

Member proxy = entityManager.getReference(Member.class, member.getId());
System.out.println(proxy.getClass());
//class practice.docs.spring.domain.Member$HibernateProxy$4giXc1wi
proxy.getName(); //호출 시점에 Member 조회 쿼리가 날아감
```

<br>

## 2. 프록시 특징

* 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
* 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다.
  * 초기화되면서 프록시 객체를 통해서 실제 엔티티에 접근이 가능하다.

> Application.java

```java
Member m1 = entityManager.find(Member.class, member.getId());
Member proxy = entityManager.getReference(Member.class, member2.getId());
System.out.println(m1.getClass() == proxy.getClass()); //false
```

* ==을 활용한 타입 비교에 주의하며 ``instanceof``를 사용하자.

> Application.java

```java
Member m1 = entityManager.find(Member.class, member.getId());
Member proxy = entityManager.getReference(Member.class, member.getId());
System.out.println(m1.getClass()); //class practice.docs.spring.domain.Member
System.out.println(proxy.getClass()); //class practice.docs.spring.domain.Member
```

* 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 ``getReference()``를 호출해도 실제 엔티티를 반환한다.

> Application.java

```java
Member proxy = entityManager.getReference(Member.class, member.getId());
Member m1 = entityManager.find(Member.class, member.getId());
System.out.println(m1.getClass()); //class practice.docs.spring.domain.Member$HibernateProxy$SCQXxq5v
System.out.println(proxy.getClass()); //class practice.docs.spring.domain.Member$HibernateProxy$SCQXxq5v
```

* 레퍼런스를 호출한 상태에서 이후 ``find()``로 엔티티를 실제로 조회하면 반대로 프록시 객체가 반환된다.
* 실제 SQL은 날아가더라도 엔티티를 반환하지 않고 프록시를 반환하는 이유?
  * JPA는 프록시든 아니든 문제가 없게 개발하는 것이 중요하다.
  * 하나의 트랜잭션에서 조회한 두 엔티티의 동일성 비교를 보장하기 위함이다.

> Application.java

```java
Member proxy = entityManager.getReference(Member.class, member.getId());
entityManager.detach(proxy);
proxy.getName();
```

* 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제가 발생한다.
  * ``getName()`` 호출시 ``org.hibernate.LazyInitializationException`` 등의 예외가 발생한다.

### 2.1. 유틸리티

> Application.java

```java
Member proxy = entityManager.getReference(Member.class, member.getId());
System.out.println(entityManagerFactory.getPersistenceUnitUtil().isLoaded(proxy)); //false
hibernate.initialize(proxy);
System.out.println(entityManagerFactory.getPersistenceUnitUtil().isLoaded(proxy)); //true
```

* 프록시 인스턴스의 초기화 여부 확인 가능하며, 강제로 초기화할 수 있다.
  * JPA 표준은 강제 초기화가 없으며, 프록시 메서드 호출을 통해 초기화가 일어난다.
  * Hibernate에서 제공한다.

<br>

## 3. 즉시 로딩과 지연 로딩

> Application.java

```java
entityManager.flush();
entityManager.clear();
Member find = entityManager.find(Member.class, member.getId());
```

* 영속성 컨텍스트가 초기화된 상태에서 Member 조회시 기본적으로 Member뿐만 아니라 Team 등 연관 객체들까지 한 꺼번에 다 조회한다.
  * ``getTeam()``이 반환하는 Team 타입의 클래스를 확인해보면 프록시가 아닌 실제 엔티티다.

> Member.java

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "team_id")
private Team team;
```

> Application.java

```java
Member find = entityManager.find(Member.class, member.getId());
System.out.println(find.getTeam().getClass()); //class practice.docs.spring.domain.Team$HibernateProxy$KlFXmep4
System.out.println("before");
find.getTeam().getName();
System.out.println("after");
```

* FetchType을 Lazy로 설정하면 지연 로딩이다.
* Member 엔티티 조회시 Team 관련 정보를 제외한 조회 쿼리를 날린다.
* Team 타입 인스턴스는 프록시 객체가 들어간다.
* 이후 Team 프록시 객체의 메서드 호출을 통해 실제 사용할 때, DB를 조회하여 Team 엔티티가 초기화된다.
  * before와 after 사이에 Team 테이블로 조회 쿼리가 날아감을 볼 수 있다.

> Application.java

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "team_id")
private Team team;
```

* Eager로 설정하면 엔티티 호출시 연관 엔티티까지 한 번에 호출해서 초기화하는 즉시 로딩을 택한다.
* JPA 구현체는 가능하면 조인 SQL을 사용해서 한번에 함께 조회한다.

<br>

## 4. 즉시 로딩 주의

* 가급적 실무에서는 지연 로딩만 사용한다.
* 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생한다.
  * 예를 들어, 엔티티 조회시 수십 개나 되는 테이블이 조인되고는 한다.
  * 실제로는 당장 사용할 필요도 없는 연관 객체들까지 조회되면 성능에 악영향이다.

> Application.java

```java
List<Member> select_m_from_member_m = entityManager.createQuery("select m from Member m")
        .getResultList();
```

> SQL

```sql
Hibernate:
    /* select
        m
    from
        Member m */ select
            member0_.id as id1_2_,
            member0_.createDate as createda2_2_,
            member0_.createdBy as createdb3_2_,
            member0_.lastModifiedBy as lastmodi4_2_,
            member0_.lastModifiedDate as lastmodi5_2_,
            member0_.name as name6_2_,
            member0_.team_id as team_id7_2_
        from
            Member member0_
Hibernate:
    select
        team0_.id as id1_4_0_,
        team0_.name as name2_4_0_
    from
        Team team0_
    where
        team0_.id=?
Hibernate:
    select
        team0_.id as id1_4_0_,
        team0_.name as name2_4_0_
    from
        Team team0_
    where
        team0_.id=?
```

* JPQL 사용시 N+1 문제가 발생하기 쉽다.
* 현재 Member 엔티티가 2개 존재하며, 각각 서로 다른 Team을 가지고 있는 상황이라고 가정해보자.
* JPQL은 주어진 쿼리문을 SQL로 번역할 뿐, 자동으로 연관 관계를 분석해서 JOIN 쿼리를 생성해 완벽한 엔티티를 반환하거나 하지는 않는다.
  * 즉, ``select m from Member m``를 ``select * from member``로 변환한다.
* JPA는 Member 정보를 다 가져왔지만, 즉시 로딩 애너테이션을 보고 Team에 대한 정보가 더 필요하다는 것을 보고 Team에 대해서도 조회 쿼리를 날린다.
  * 영속성 컨텍스트에 공유되는 Team이 아니기에 Team 테이블에 쿼리를 2번 날려서 조회해온다.
* N+1 문제란 쿼리를 1개 날렸는데 추가 쿼리 N개가 더 나가는 문제를 일컫는다.
  * Lazy로 쓰면 이러한 문제가 어느정도 해소된다.
  * 후술할 Fetch Join 등으로 해결한다.
* @ManyToOne, @OneToOne은 기본이 즉시 로딩 이기에 별도로 Lazy를 설정해야 한다.
  * 반면 @OneToMany, @ManyToMany는 기본이 지연 로딩이다.

<br>

## 5. 영속성 전이 Cascade

> Parent.java

```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;
    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }
}
```

> Child.java

```java
@Entity
public class Child {

    @Id
    @GeneratedValue
    private Long id;
    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
}
```

> Application.java

```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();
parent.addChild(child1);
parent.addChild(child2);

entityManager.persist(child1);
entityManager.persist(child2);
entityManager.persist(parent);
```

* 위와 같은 경우 일일이 Child에 대해서도 전부 영속화시켜야 한다.

> Parent.java

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
private List<Child> childList = new ArrayList<>();
```

* 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만든다.
  * ``persist()`` 메서드를 Parent에 대해서만 호출해도 자동으로 Child까지 영속화된다.
* 영속성 전이는 연관 관계를 매핑하는 것과 아무 관련이 없다.
  * 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다.
* 옵션은 Persist, Remove, ALL 등이 있으며 ALL은 이 모든 옵션들을 제공한다.
  * 즉, 위에서는 ``remove()``로 Parent 엔티티를 삭제하면 Child 엔티티들도 함께 삭제 쿼리가 나간다.
  * 다만 해당 옵션에서는 Child 엔티티를 컬렉션에서 제거한다고 자동으로 Child 엔티티가 삭제되지는 않는다.

### 5.1. 고아 객체 제거

> Parent.java

```java
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> childList = new ArrayList<>();
```

> Application.java

```java
Parent parent1 = entityManager.find(Parent.class, parent.getId());
parent1.getChildList().remove(0);
```

* 부모 엔티티와 연관 관계가 끊어진 자식 엔티티를 자동으로 삭제해준다.
* **자식 엔티티를 컬렉션에서 제거하면 자동으로 해당 자식 엔티티에 대한 삭제 쿼리가 나간다.**
* 개념적으로 부모를 제거하면 자식은 고아가 된다.
  * 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다.
  * CascadeType.REMOVE처럼 동작한다.

### 5.2. 주의점

* 두 옵션 모두 부모가 1개이고 자식에 대한 단일 소유자일 때만 사용하도록 주의한다.
  * 즉, 참조하는 곳이 부모 하나일 때만 사용해야 한다.
  * 다른 엔티티에서도 해당 자식들을 소유하거나 참조하는 경우 해당 옵션 사용을 지양한다.
* 부모와 자식의 라이프사이클이 거의 유사한 경우 사용한다.
  * 즉, 게시글과 첨부 파일의 관계와 같이.
* @OneToOne, @OneToMany만 사용이 가능하다.

### 5.2. 생명 주기

* CascadeType.ALL + orphanRemovel=true를 사용하는 경우?
* 스스로 생명주기를 관리하는 엔티티는 ``persist()``로 영속화하며, ``remove()``로 제거한다.
* 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명 주기를 관리할 수 있다.
* 도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때 유용하다.
  * Root는 Parent가 되며, 관리할 Aggregate는 Child들이 된다.
  * Child들에 대한 Repository를 구현할 필요가 없어진다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
