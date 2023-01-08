---
title: "[Java ORM 표준 JPA 프로그래밍] 9장 : 값 타입"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-27
last_modified_at: 2021-06-27
---

## 1. JPA 데이터 타입 분류

* 엔티티 타입
  * @Entity로 정의하는 객체다.
  * 데이터가 변경되어도 식별자로 지속해서 추적이 가능하다.
* 값 타입
  * int, Integer, String처럼 단순히 값으로 사용하는 Java 기본 타입이나 객체다.
  * 식별자가 없고 값만 있으므로 변경시 추적이 불가능하다.
  * 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체된다.

<br>

## 2. 값 타입 분류

* 기본값 타입
  * Java 기본 타입(int, double)
  * 래퍼 클래스(Integer, Long)
  * String
* 임베디드 타입(복합 값 타입)
* 컬렉션 값 타입

<br>

## 3. 기본값 타입

* ``String name``과 같은 타입은 생명 주기를 엔티티에 의존한다.
  * 회원을 삭제하면 이름, 나이 등의 필드도 함께 삭제된다.
* 값 타입은 공유하면 안 된다.
  * 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안 된다.
* int 등 Java의 기본 타입은 항상 값을 복사하기 때문에 절대 공유되지 않는다.
  * Integer같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능하지만 불변이다.

<br>

## 4. 임베디드 타입

* JPA는 새로운 값 타입을 직접 정의할 수 있으며, 이를 임베디드 타입이라고 한다.
* 주로 기본값 타입을 모아서 만들기에 복합값 타입이라고도 한다.

> Period.java

```java
@Embeddable
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;
}
```

> Member.java

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "team_id")
    private Team team;
    @Embedded
    private Period period;
    @Embedded
    private Address address;
}
```

* 기본 생성자가 필수다.
* 임베디드 타입은 재사용이 가능하며 Period 및 Address는 각각 고유의 비즈니스 로직을 가질 수 있다.
* 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존한다.
* 임베디드 타입은 엔티티의 값일 뿐이다.
  * Period 및 Address 관련 데이터는 별도의 테이블을 생성해서 저장하는 것이 아니라, Member 테이블에 전부 저장되어 있다.
  * 단지 Java 상에서 주소 및 기간 관련 데이터들을 별도의 클래스로 분리해 응집도 높게 사용할 수 있다.
  * 임베디드 사용 전이나 후나 맵핑되는 테이블은 같다.
* 객체와 테이블을 아주 세밀하게 맵핑하는 것이 가능하다.
* 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.

### 4.1. 칼럼 중복

> Member.java

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String name;
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "team_id")
    private Team team;
    @Embedded
    private Period period;
    @Embedded
    @AttributeOverrides({@AttributeOverride(name = "street", column = @Column(name = "another_street")),
            @AttributeOverride(name = "district", column = @Column(name = "another_district")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "another_zipcode"))})
    private Address anotherAddress;
    @Embedded
    private Address address;
}
```

* 한 엔티티에서 여러 개의 같은 값 타입을 사용하면 칼럼 명이 중복된다.
* @AttributeOverride를 통해 칼럼명 속성을 재정의해준다.
* 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null이 된다.

### 4.2. 불변

> Application.java

```java
Address address = new Address();

Member member = new Member();
member.setAddress(address);

Member member2 = new Member();
member2.setAddress(address);

entityManager.persist(member);
entityManager.persist(member2);

member.getAddress().setZipcode("changed");
```

* 임베디드 타입같은 값 타입을 여러 엔티티에서 공유하면 위험하다.
  * 의도는 member만 수정하는 것이지만, 같은 Address를 참조하는 member2에 대해서도 Update 쿼리가 나간다.
* 값 타입의 실제 인스턴스인 값을 공유하면 부작용이 발생한다.
* 따라서 값 타입을 불변 객체로 설계함으로써 이러한 부작용을 막는다.
  * 혹은 정말 setter를 사용해야 한다면 member2에 address를 주입할 때 **방어적 복사**를 수행한다.

### 4.3. 비교

* 값 타입은 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 인식한다.
* 동일성 비교란 인스턴스의 참조값을 ==을 통해 비교한다.
* 동등성 비교란 ``equals()`` 등을 통해 인스턴스의 값을 비교한다.
* 값 타입 비교시 ``equals()``와 ``hashcode()``를 재정의하자!

<br>

## 5. 값 타입 컬렉션

> Member.java

```java
@ElementCollection(fetch = FetchType.LAZY)
@CollectionTable(name = "FAVORIATE_FOOD", joinColumns = {@JoinColumn(name = "member_id")})
@Column(name = "food_name")
private Set<String> foods = new HashSet<>();

@ElementCollection(fetch = FetchType.LAZY)
@CollectionTable(name = "ADDRESS_HISTORY", joinColumns = {@JoinColumn(name = "member_id")})
private List<Address> histories = new ArrayList<>();
```

* 값 타입을 하나 이상 저장할 때 사용한다.
  * @ElementCollection, @CollectionTable을 사용한다.
* DB는 컬렉션을 같은 테이블에 저장할 수 없으며, 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.
* 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.
  * 생명 주기가 엔티티에 의존한다.
  * 별도로 값 컬렉션을 ``persist()``하지 않아도 상위 엔티티를 영속화할 때 함께 영속화된다.
  * 값 컬렉션에 변화가 생겨도 자동으로 수정된다.
* 값 타입 컬렉션도 기본적으로 지연 로딩 전략을 사용한다.

### 5.1. 제약 사항

> Application.java

```java
Member find = entityManager.find(Member.class, member.getId());
find.getHistories().remove(new Address("aaa", "bbb", "ccc"));
find.getHistories().add(new Address("aab", "bbb", "ccc"));
```

> SQL

```sql
Hibernate:
    select
        histories0_.member_id as member_i1_0_0_,
        histories0_.district as district2_0_0_,
        histories0_.street as street3_0_0_,
        histories0_.zipcode as zipcode4_0_0_
    from
        ADDRESS_HISTORY histories0_
    where
        histories0_.member_id=?
Hibernate:
    /* delete collection practice.docs.spring.domain.Member.histories */ delete
        from
            ADDRESS_HISTORY
        where
            member_id=?
Hibernate:
    /* insert collection
        row practice.docs.spring.domain.Member.histories */ insert
        into
            ADDRESS_HISTORY
            (member_id, district, street, zipcode)
        values
            (?, ?, ?, ?)
Hibernate:
    /* insert collection
        row practice.docs.spring.domain.Member.histories */ insert
        into
            ADDRESS_HISTORY
            (member_id, district, street, zipcode)
        values
            (?, ?, ?, ?)
```

* histories 값 컬렉션에 원소 2개가 존재하고, 한 원소의 내용을 변경하는 예제다.
* 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
* 값 타입은 엔티티와 다르게 식별자 개념이 없다.
  * 값은 변경하면 추적이 어렵다.
* 그렇다고 값 타입 컬렉션을 매핑하는 테이블에 PK를 주려면?
  * 모든 컬럼을 묶어서 기본 키를 구성해야 한다.
  * null 입력 및 중복 저장을 금지한다.

### 5.2. 대안

> Member.java

```java
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "member_id")
private List<AddressEntity> histories = new ArrayList<>();
```

> AddressEntity.java

```java
@Entity
public class AddressEntity {

    @Id
    @GeneratedValue
    private Long id;
    private Address address;
}
```

* 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려한다.
  * 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용한다.
* 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용한다.

### 5.3. 정리

* 엔티티 타입의 특징
  * 식별자가 있다.
  * 생명 주기 관리가 되며, 공유할 수 있다.
* 값 타입의 특징
  * 식별자가 없다.
  * 생명 주기가 엔티티에 의존한다.
  * 공유하지 않는 것이 안전하며, 복사해서 사용해야 한다.
  * 불변 객체를 고려한다.
* 값 타입은 정말 값 타입이라 판단될 때만 사용한다.
  * 식별자가 필요하고, 지속해서 값을 추적 및 변경해야 한다면 엔티티다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
