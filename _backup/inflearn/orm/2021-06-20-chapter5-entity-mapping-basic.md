---
title: "[Java ORM 표준 JPA 프로그래밍] 5장 : 연관 관계 매핑 기초"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-20
last_modified_at: 2021-06-20
---

## 1. DB 종속적인 테이블 설계

> Application.java

```java
Team team = new Team(); 
team.setName("TeamA"); 
em.persist(team); 
 
Member member = new Member(); 
member.setName("member1"); 
member.setTeamId(team.getId()); 
em.persist(member); 

//..
Member member = em.find(Member.class, 3L);
Team team = em.find(Team.class, member.getTeamId());
```

* 객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다.
* 현재 방식은 객체 설계를 테이블 설계에 맞춘 방식이다.
  * 테이블의 외래키를 객체에서 그대로 가져온다.
  * 객체 그래프 탐색이 불가능하다.
  * 참조가 없으므로 UML또한 잘못 되었다.
* 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
  * 테이블은 외래 조인을 사용해서 연관된 테이블을 찾는다.
  * 객체는 참조를 통해서 연관된 객체를 찾는다.
  * 테이블과 객체 사이에는 이런 큰 간격이 있다.
* Member 엔티티가 getter를 통해 자유롭게 연관 관계를 맺은 참조 객체를 가져올 수는 없을까?

<br>

## 2. 단방향 연관 관계 맵핑

> Member.java

```java
@Entity
public class Member {

    @Id
    @Column(name = "member_id")
    @GeneratedValue
    private Long id;
    private String name;

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

* Member와 Team은 N:1 관계를 형성한다.
* Member에서 @JoinColumn을 통해 다른 엔티티 테이블의 어떤 칼럼과 조인하는지 참조 FK(외래키)를 명시한다.

> Application.java

```java
//팀 저장 
Team team = new Team(); 
team.setName("TeamA"); 
em.persist(team); 
 
Member member = new Member(); 
member.setName("member1"); 
member.setTeam(team);
em.persist(member);

//..
Member findMember = em.find(Member.class, 3L);
Team findTeam = findMember.getTeam();
```

* 객체 그래프 탐색을 통해 참조로 자유롭게 연관 관계를 조회한다.
* setter를 통해 Member의 Team을 수정하면 커밋 시점에 참조하는 Team의 ID PK가 변경된다.

<br>

## 3. 양방향 맵핑

> Team.java

```java
@OneToMany(mappedBy = "team")
private List<Member> members = new ArrayList<>();
```

> Application.java

```java
Member findMember = entityManager.find(Member.class, member.getId());
List<Member> members = findMember.getTeam().getMembers();

members.forEach(t -> System.out.println(t.getName()));
```

* Team 엔티티에서 역방향으로 Member 객체 그래프를 탐색할 수 있다.

### 3.1. 패러다임 차이

* DB 상의 Member 테이블과 Team 테이블은 연관 관계가 1개이며 양방향으로 접근이 가능하다.
* Member 테이블에 있는 TEAM_ID 외래키 하나로 두 테이블의 연관 관계를 관리할 수 있다.
  * 즉, 양방향 연관 관계란 양쪽으로 조인이 가능하다.
* 반면 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단뱡향 관계가 2개인 것이다.
  * 객체를 양방향으로 참조하려면 단방향 연관 관계가 두 개 존재햐아 한다.
  * 예를 들어, A 클래스는 내부적으로 B 인스턴스를 참조하고 B 클래스는 내부적으로 A 인스턴스를 참조하는 것이다.

### 3.2. 연관 관계의 주인(Owner)

![image](https://user-images.githubusercontent.com/56240505/122664800-7cb34380-d1de-11eb-91be-0191aeaa960c.png)

* 양방향 맵핑에서는 객체의 두 관계중 하나를 연관 관게의 주인으로 지정해야 한다.
  * Member 엔티티 객체에 팀을 ``setter``로 주입했을 때 Member 테이블 FK가 변경되어야 하는가?
  * 혹은 특정 Team의 엔티티가 관리하는 Member 컬렉션에 Member를 추가했을 때 테이블의 FK가 변경되어야 하는가?
* 연관 관계의 주인만이 외래 키를 등록 및 수정 등 관리할 수 있다.
* 주인이 아닌 쪽은 읽기만 가능하다.
* 주인은 ``mappedBy`` 속성을 사용하지 않고, 주인이 아니면 ``mappedBy`` 속성으로 주인을 지정해준다.
  * 주로 외래 키가 있는 곳을 주인으로 정한다.
  * N:1 맵핑의 경우 N 테이블 쪽에 FK가 존재하기에 Member 엔티티를 주인으로 관리하는 것이 편하다.
    * Team 엔티티를 주인으로 만들 수 있으나 Team 엔티티를 수정했을 때 Member 테이블에 쿼리가 날아가는 등 직관적이지 않다.

### 3.3. 주의사항

> Application.java

```java
Team team = new Team(); 
team.setName("TeamA"); 
em.persist(team); 
 
Member member = new Member(); 
member.setName("member1"); 
team.getMembers().add(member); 
 em.persist(member); 
```

* 역방향만 연관 관계를 설정하고 연관관계의 주인에 값을 입력하지 않았다.
* 실제 DB를 조회하는 경우 Member 테이블의 TEAM_ID는 null이 된다.
  * 영속화 전에 ``member.setTeam(team);``를 호출해야 한다.

> Application.java

```java
Team team = new Team();
team.setName("black team");
entityManager.persist(team);

Member member = new Member();
member.setName("abc");
member.setTeam(team);
//team.getMembers().add(member); 
entityManager.persist(member);

//만약 여기서 flush, clear 등을 호출한다면 isEmpty()는 false다

Member findMember = entityManager.find(Member.class, member.getId());
System.out.println(findMember.getTeam().getMembers().isEmpty()); //true
transaction.commit();
```

* Team과 Member는 ``persist()``를 통해 영속성 컨텍스트의 1차 캐시에 존재한다.
  * 추후 Member 조회를 요청했을 때 실제 DB에서 엔티티를 조회하는 것이 아니라 1차 캐시에서 가져온다.
  * 영속화 전에 ``team.getMembers().add(member); ``를 호출하지 않아 역방향 조회시 컬렉션이 비어있다.
* 만약 Member 조회 이전에 플러시하고 영속성 컨텍스트를 초기화한 상태라면 위 경우 DB에서 새로 Member를 조회하기 때문에 문제가 되지 않는다.
  * 지연 로딩을 사용하기에 ``findMember.getTeam().getMembers()`` 호출을 통해 실제 Team 엔티티 내부의 컬렉션 객체가 필요한 순간 JOIN 조회 쿼리가 날아간다.
* 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자.

> Team.java

```java
public void updateTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

* 매번 setter를 호출하다 보면 실수하기 쉬우니 연관 관계를 위한 편의 메서드를 만든다.
* 양방향 매핑시 무한 루프를 조심해야 한다.
  * Lombok 등을 통해 Member와 Team 엔티티에 ``toString()``을 아무 생각없이 만들면?
  * ``toString()`` 호출시 무한 루프에 빠진다.
    * 특히 Controller와 View 사이에서 주고 받는 데이터 타입을 엔티티로 직접 사용하는 경우 이와 같은 문제가 발생하기 쉽다.
    * 메시지 컨버터가 JSON과 Java 객체 사이를 변환할 때 ``toString()``을 호출하기 때문이다.
    * 따라서 표현 계층에서는 Entity를 직접 사용하기 보다는 DTO를 사용하자.
      * 또한 Entity를 DTO 용도로 함께 사용하면 Entity 변경시 API 스펙이 변경된다.

### 3.4. 정리

* 단방향 매핑만으로도 이미 연관관계 매핑은 완료된 것이다.
* 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것일 뿐이다.
* JPQL에서 역방향으로 탐색할 일이 많다.
* 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 된다.
  * 양방향 매핑 설정은 Java에서의 사용을 위함이지 실제 테이블에는 영향을 주지 않는다.
  * 단방향이 고려해야할 점이나 복잡도가 낮기 때문이다.
* 비즈니스 로직을 기준으로 연관 관계의 주인을 선택하면 안 된다.
* 연관관계의 주인은 외래 키의 위치를 기준으로 정해야 한다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
