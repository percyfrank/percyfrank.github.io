---
title: "[Java ORM 표준 JPA 프로그래밍] 6장 : 다양한 연관 관계 매핑"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-20
last_modified_at: 2021-06-20
---

## 1. 다중성

* @ManyToOne
* @OneToMany
* @OneToOne
* @ManyToMany

### 1.1. 테이블과 객체

* 테이블은 외래키 하나로 양방향 조인이 가능하다.
  * 사실 방향이라는 개념이 없다.
* 반면 객체는 참조용 필드가 있는 쪽으로만 참조가 가능하다.
  * 한 쪽만 참조하면 단방향이며 양쪽이 서로 참조하면 양방향이다.

### 1.2. 연관 관계의 주인

* 테이블은 외래키 하나로 두 테이블이 연관 관계를 맺는다.
* 객체 양방향 관계는 A->B, B->A 처럼 참조가 2군데 존재한다.
  * 둘 중 테이블의 외래키를 관리할 곳을 지정해야 한다.
* 연관 관계의 주인은 외래키를 관리하는 참조이며, 주인의 반대편은 외래키에 영향을 주지 않고 단순 조회만 가능하다.

<br>

## 2. 다대일(N:1)

### 2.1. 다대일 단방향/양방향

![image](https://user-images.githubusercontent.com/56240505/122667548-4da4ce00-d1ee-11eb-9e8e-50ac182f70cb.png)

> Member.java

```java
@Entity
public class Member {

    @Id
    @Column(name = "member_id")
    @GeneratedValue
    private Long id;
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

> Team.java

```java
@Entity
public class Team {

    @Id
    @Column(name = "team_id")
    @GeneratedValue
    private Long id;
    private String name;
}
```

* 가장 많이 사용하는 연관 관계이며, 다대일의 반대는 일대다다.
* **외래 키가 있는 쪽이 연관 관계의 주인이다.**

> Team.java

```java
@OneToMany(mappedBy = "team")
private List<Member> mebers = new ArrayList<>();
```

* 필요하다면 양쪽을 서로 참조하도록 개발한다.

<br>

## 3. 일대다(1:N)

### 3.1. 일대다 단방향

![image](https://user-images.githubusercontent.com/56240505/122667695-17b41980-d1ef-11eb-8a44-82cce86b2dae.png)

> Team.java

```java
@Entity
public class Team {

    @Id
    @Column(name = "team_id")
    @GeneratedValue
    private Long id;
    private String name;
    @OneToMany
    @JoinColumn(name = "team_id")
    private List<Member> mebers = new ArrayList<>();
}
```

* 일대다 단방향은 일대다(1:N)에서 일(1)이 연관 관계의 주인이 된다.
* 테이블 일대다 관계는 항상 다(N)쪽에 외래키가 있다.
  * 다대일(N:1)과 DB 스키마 차이는 없다.
* 객체와 테이블의 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조다.
* @JoinColumn을 반드시 사용해야 하며, 그렇지 않으면 중간에 테이블을 하나 추가하는 조인 테이블 방식을 사용한다.

> Application.java

```java
Member member = new Member();
member.setName("abc");

entityManager.persist(member);
Team team = new Team();
team.setName("team");
team.getMebers().add(member);
entityManager.persist(team);
```

* 엔티티가 관리하는 외래키가 다른 테이블에 있다는 단점이 존재한다.
  * 연관관계 관리를 위해 추가로 UPDATE SQL가 실행된다.
  * Team 엔티티를 영속화하는 시점에 Team Insert 쿼리 뿐만 아니라, Member 테이블의 기존 Member 엔티티에 대한 TEAM_ID PK Update 쿼리도 날아간다.
  * 가독성이 좋은 편이 아니고 오해할 여지가 크다.
* 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자.
  * 객체적으로 깔끔하지 않고 약간의 Trade-Off가 있을지라도 말이다.

### 3.2. 일대다 양방향

![image](https://user-images.githubusercontent.com/56240505/122668008-fa804a80-d1f0-11eb-9851-e90e53e4ad19.png)

> Member.java

```java
@ManyToOne
@JoinColumn(insertable = false, updatable = false)
private Team team;
```

* 일대다 양방향을 지정하려면 두 곳 모두 @JoinColumn을 사용해야 한다.
  * 양쪽 모두 연관 관계의 주인이 되기 때문에 관리하지 않는 쪽에서 특수 속성을 통해 쓰기를 금하고 일기 전용으로 만든다
* 일대다 양방향은 공식적으로 존재하지 않고 읽기 전용 필드를 사용해 양방향처럼 사용하는 것이다.
* **그냥 다대일 양방향을 사용하자.**

<br>

## 4. 일대일(1:1)

### 4.1. 주 테이블에 외래키 단방향/양방향

![image](https://user-images.githubusercontent.com/56240505/122668129-99a54200-d1f1-11eb-8fd4-7157e324115d.png)

* 일대일 관계는 그 반대도 일대일이다.
* 주 테이블이나 대상 테이블 중 어디에 외래키를 넣을지 선택이 가능하다.
* 외래키에 데이터베이스 유니크(UNI) 제약 조건을 추가해야 한다.

> Member.java

```java
@OneToOne
@JoinColumn(name = "locker_id")
private Locker locker;
```

* 다대일(@ManyToOne) 단방향 매핑과 유사하다.

> Locker.java

```java
@OneToOne(mappedBy = "locker")
private Member member;
```

* 양방향을 만들고 싶다면?
  * 다대일 양방향 매핑처럼 외래키가 있는 곳이 연관 관계의 주인이 된다.
  * 반대편인 Locker는 칼럼을 추가하고 ``mappedBy`` 속성을 지정해준다.

### 4.2. 대상 테이블에 외래키 단방향

![image](https://user-images.githubusercontent.com/56240505/122668290-4e3f6380-d1f2-11eb-97cd-06af92219c91.png)

* 단방향 관계는 JPA에서 지원하지 않으며, 양방향 관계는 가능하다.

### 4.3. 대상 테이블에 외래키 양방향

![image](https://user-images.githubusercontent.com/56240505/122668341-78912100-d1f2-11eb-8648-bdd82073f560.png)

* 기존 일대일 테이블을 거꾸로 뒤집은 것이다.
* 일대일 주 테이블에 외래키 양방향과 매핑 방법은 같다.
* 외래키를 Member에 넣든 Locker에 넣든 일대일 관계는 유효하지만, 미래의 비즈니스 로직 변경 여부 등에 따라 어느 쪽에 외래키가 위치해야 할지 고민해봐야 한다.

### 4.4. 정리

* 주 테이블에 외래키 존재?
  * 주 객체가 대상 객체의 참조를 가지는 것처럼,  주 테이블에 외래 키를 두고 대상 테이블을 찾는다.
  * 객체지향 개발자가 선호하며 JPA 매핑이 편리하다.
  * 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인이 가능하다.
  * 단점 : 값이 없으면 외래키에 null을 허용해야 한다.
* 대상 테이블에 외래키 존재?
  * 전통적인 데이터베이스 개발자가 선호한다.
  * 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지가 가능하다.
  * 단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩되어야 한다.
* 지연 로딩으로 설정했을 때 Member의 연관 엔티티 Locker가 있으면 프록시 객체가 들어가고, 아니면 null이 들어간다.
  * Member 테이블에 Locker FK가 있으면 값이 있는지 없는지 주 테이블을 로드하는 시점에서 FK 컬럼을 보고 확인하기 때문에 지연 로딩이 가능하다.
  * 반면 Locker가 Member FK를 가지면 Member 테이블 로드 시점에 이를 알수가 없고, 결국에는 Member가 보유한 Locker가 있는지 없는지 확인하기 위해 Locker 테이블에 대한 조회 쿼리가 들어가야 한다.
  * 즉, 지연 로딩을 위해 프록시를 넣을지 null을 넣을지 판단해야 하는데 어차피 쿼리를 날려서 조회해야 하기 때문에 지연 로딩이 쓸데가 없다.

<br>

## 5. 다대다(N:M)

![image](https://user-images.githubusercontent.com/56240505/122668827-f8b88600-d1f4-11eb-8223-61a75aa4d8b7.png)

* 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
* 연결 테이블을 추가해서 일대다 혹은 다대일 관계로 풀어내야 한다.
* 반면 객체는 컬렉션을 통해 객체 2개로 다대다 관계 표현이 가능하다.

> Member.java

```java
@ManyToMany
@JoinTable(name = "MEMBER_PRODCUT")
private List<Product> products = new ArrayList<>();
```

* 단방향을 지정할 때 @JoinTable을 설정해야 한다.
  * 두 테이블을 연결하는 중간 테이블을 생성한다.

> Product.java

```java
@ManyToMany(mappedBy = "products")
private List<Members> members = new ArrayList<>();
```

* 양방향 설정시 반대쪽에 ``mappedBy`` 속성을 추가해준다.

### 5.1. 단점

* 편리해 보이지만 실무에서 사용하기 어렵다.
* 실무에서는 두 테이블을 연결할 때, 연결 테이블이 단순히 연결면 하지 않고 그 중간 테이블에 주문 시간이나 수량 등 다양한 비즈니스 관련 데이터가 들어오는 일이 빈번하다.
  * 그러나 @ManyToMany로 생성하는 연결 테이블에는 그 외 비즈니스 데이터를 추가할 수 없다.
  * 실제 비즈니스는 @ManyToMany로 풀 수 있는 경우가 많지 않다.

![image](https://user-images.githubusercontent.com/56240505/122668965-e428bd80-d1f5-11eb-986f-3037d1b4b703.png)

* 연결 테이블용 엔티티를 추가함으로써 연결 테이블을 엔티티로 승격시키면 한계를 극복할 수 있다.
  * @ManyToMany -> @OneToMany, @ManyToOne
* 왠만하면 중간 연결 테이블 PK는 비즈니스 로직이 없는 의미 없는 값으로 지정하고, 자동 생성이 되도록 한다.
  * 기존의 연결 테이블은 MEMBER_ID와 PRODUCT_ID가 모두 PK와 FK로 잡혀있다.
    * MEMBER와 PRODUCT가 한데 묶여서 들어갈 수 있다는 의미가 생겨난다.
    * 개발할 때는 문제가 없으나, ID가 어디에 종속되어있으면 시스템을 추후 변경할 때 어려움이 있을 수 있다.
  * 따라서 승격된 중간 연결 테이블을 ORDER_ID라는 별도의 PK를 가지고 MEMBER_ID와 PRODUCT_ID는 FK로 내린다.

<br>

## 6. 실전 예제

* 테이블의 N:M 관계는 지양하고, 중간 테이블을 이용해서 1:N, N:1로 변환한다.
  * 실전에서는 중간 테이블이 단순하지 않다.
* @ManyToMany는 제약이 많다.
  * 필드 추가가 안 되며, 엔티티 테이블이 불일치한다.
* 각 연관 관계 맵핑 애너테이션의 세부 속성은 필요할 때마다 자료를 찾아보자.

> Cateogry.java

```java
@Entity
public class Category {

    @Id
    @Column(name = "category_id")
    @GeneratedValue
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    @ManyToMany
    @JoinTable(name = "CATEOGRY_ITEM", joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();

}
```

* Category끼리의 연관 관계를 맵핑함으로써 부모-자식 관계를 형성할 수 있다.

> Item.java

```java
@Entity
public class Item {

    @Id
    @Column(name = "item_id")
    @GeneratedValue
    private Long id;
    private String name;
    private Long price;
    private Long stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories;
}
```

> SQL

```sql
create table Category (
   category_id bigint not null,
    name varchar(255),
    parent_id bigint,
    primary key (category_id)
)

create table CATEOGRY_ITEM (
   category_id bigint not null,
    item_id bigint not null
)
```

* Cateogry와 Item간의 다대다 연관 관계 맵핑을 하면 연결 테이블이 생성된다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
