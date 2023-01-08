---
title: "JDBC vs SQL Mapper vs ORM"
excerpt: "Persistence를 관리하는 방법들에 대해 알아보자."
categories:
  - Spring Data
tags:
  - Spring Data
date: 2021-06-08
last_modified_at: 2021-06-08
---

## 1. Persistence

Persistence(영속성)이란 데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성을 의미한다.

### 1.1. JDBC

Java 진영에서는 [JDBC](https://xlffm3.github.io/java/jdbc/)를 통해 Persistence를 쉽게 구현할 수 있다.

클라이언트는 JDBC API를 통해 DB 벤더에 상관없는 일관된 DB 액세스 로직을 처리할 수 있다. 그러나 연결을 맺고 반환하는 등 중복되는 보일러플레이트 코드가 발생한다. 또한 커넥션을 제대로 반환하지 못하면 문제가 발생한다. 아울러, DB마다 상이한 정보를 가진 확인 예외(SQLException)을 처리해야 한다.

### 1.2. Persistence Framework

순수한 JDBC API를 사용하는 것은 복잡하고 번거롭다. Persistence Framework는 간단하게 DB 연동 작업을 수행한다. 이러한 Persistence Framework는 내부적으로 JDBC API를 사용한다.

그 종류로는 SQL Mapper, ORM 등이 있다.

<br>

## 2. SQL Mapper

SQL을 직접 작성하며, SQL 쿼리 결과와 객체의 필드를 맵핑하여 데이터를 객체화한다. 대표적으로 Spring JDBC 및 MyBatis 등이 있다. Spring JDBC는 쿼리 수행 결과와 객체의 필드를 맵핑하며, RowMapper를 재활용할 수 있다. 템플릿-콜백 패턴을 통해 JDBC API 사용시 반복적으로 발생하는 작업을 대신 해준다.

### 2.1. MyBatis

MyBatis는 SQL 쿼리들을 XML 파일에 작성함으로써 DAO 코드와 SQL을 나눠서 관리한다. JDBC를 사용하면 쿼리 결과를 객체의 인스턴스에 맵핑하기 위한 코드가 길어지는데, MyBatis는 이를 줄일 수 있다. 또한 조건에 따른 분기 등 쿼리를 동적으로 작성할 수 있다.

정리하자면 반복적인 JDBC 프로그래밍을 단순화하면서 동시에, 복잡하거나 동적인 쿼리를 작성할 수 있고, DAO로부터 SQL라는 관심사를 분리함으로써 코드 간결성 및 유지 보수성을 높인다.

### 2.2. 한계

SQL을 직접 다룰 때 개발 사이클이 다음과 같이 진행되기 쉽다.

1. 테이블 설계
2. 테이블로부터 객체 도출
3. 객체와 테이블 매핑
4. DB 접근 로직 구현
5. 비즈니스 로직 구현
6. 객체 속성 추가 및 테이블 컬럼 추가

SQL 의존적인 개발로 인해, 비즈니스 로직 구현보다 스키마 및 접근 로직 등의 DB 관련 설계가 중요해진다. 또한 특정 DB에 종속되기 마련이다. DB마다 비표준적인 SQL 문법을 사용하며, 테이블 변경시 SQL 쿼리 및 객체 필드 모두 수정이 필요하다.

**코드상으로는 SQL과 JDBC API를 분리했더라도, 논리적으로는 강한 의존성을 가지고 있다.**

<br>

## 3. 패러다임 불일치

OOP는 추상화, 상속, 다형성 등을 활용하며 기능과 속성을 한 곳에서 관리하는 기술이다. 반면 RDB는 어떻게 데이터를 저장할지에 초점이 맞추는 데이터 중심의 기술이다.

각각의 지향 목적이 다르기 때문에 사용 방법과 표현 방식에 있어 차이가 존재한다. SQL Mapper는 객체 모델링보다는 테이블 모델링에만 집중하며, 객체는 단순히 테이블에 맞추어 데이터를 전달하는 역할로 전락하게 된다.

> Crew.java

```java
class Crew {

    Long id;
    String nickName;
    Team team;
}

class Team {

    Long id;
    Strin name;
}
```

* 객체는 참조를 통해 다른 객체들과 연관 관계를 가진다.
* 문제는 패러다임 불일치로 인해 이러한 객체를 DB에 저장하고 조회하는 것이 어렵다.

> Crew.java

```java
class Crew {

    Long id;
    Long teamId; //TEAM_ID FK 컬럼 사용
    Team team;
}

class Team {

    Long id;
    Strin name;
}
```

* RDB 테이블은 외래키를 사용해 다른 테이블과 연관 관계를 가진다.
* 이러한 테이블 모델링에 맞춰 객체를 모델링한다면 객체지향의 이점이 사라지고 DB 의존적이게 된다.

> Main.java

```java
//객체지향적인 코드
Crew crew = crewDao.findCrew();
Team team = crew.getTeam();

//RDB의 문제
Crew crew = crewDao.findCrew();
Team team = teamDao.findTeam(crew.getTeamId());
```

* Crew와 Team이 어떤 관계인지 알기 모호해진다.

<br>

## 4. ORM

ORM(Object Relational Mapping)은 객체와 관계형 DB를 맵핑하는 기술이다. SQL 쿼리가 아닌 직관적인 메서드 코드로 데이터를 조작한다. ORM은 객체간의 관계를 바탕으로 SQL을 자동 생성하고 메서드로 이를 조작하도록 한다.

JPA는 Java 진영의 ORM에 대한 API 표준 명세이다. JPA 인터페이스 구현체 프레임워크로는 Hibernate, Eclipse Link 등이 있다.

### 4.1. 상속 문제

객체에는 상속이란 개념이 존재하지만, RDB는 그렇지 않다.

> Person.java

```java
class Person {

    Long id;
    String name;
}

class Crew extends Person {

    String nickName;
}
```

> Query

```sql
insert into person ...;
insert into crew ...;

select p.*, c.*
from person p join crew c
on p.id = c.id;
```

* 이를 RDB 테이블로 표현하려면, Person 테이블과 Crew 테이블이 모두 존재해야 한다.
* 슈퍼타입 및 서브타입에 대한 테이블을 각각 만들어서 데이터 삽입 및 조회가 가능하지만, 여러 번 쿼리를 날리거나 테이블 조인하는 등 복잡한 작업이 수반된다.

> Main.java

```java
Person crew = jpa.find(Crew.class, crewId);
```

* JPA는 이러한 복잡한 쿼리를 대신 날려준다.

### 4.2. 객체 그래프 탐색 문제

객체지향적으로 모델링된 객체들은 테이블로 CRUD 하기 어려워진다. 그러나 JPA는 객체의 참조로 형성된 연관 관계를 간편하게 저장 및 조회할 수 있다.

> Main.java

```java
crew.setTeam(team);
jpa.persist(crew);

Crew crew = jpa.find(Crew.class, id);
Team team = crew.getTeam();
```

기존 JDBC API를 사용하는 방식은 엔티티를 신뢰하며 사용할 수 없다. 과연 ``crewDao.findCrew()``는 Crew가 참조하는 Team 객체를 반환한다고 보장할 수 있는가? 조인, 조건 쿼리를 제대로 작성하지 않으면 null이 반환될 수 있다. 이처럼 객체 내에서는 그래프 탐색이 자유롭지만, SQL을 다룰 때는 작성 쿼리에 따라 객체 그래프 범위가 제한된다.

반면 JPA는 반환된 엔티티를 신뢰하며 사용할 수 있다. JPA는 Team의 참조를 자동으로 외래키로 변환해 쓰기 및 읽기 쿼리를 생성하여 작업을 수행한다. 연관 객체를 조회하고, 그에 필요한 적절한 쿼리를 생성 및 호출하여 완벽한 엔티티로 객체화하기 때문이다.

### 4.3. 비교 문제

DB는 기본키 값으로 각각의 Row를 비교한다. 반면 OOP는 동일성 및 동등성 비교로 인스턴스를 구분한다. DAO의 조회 메서드는 항상 new 연산을 통해 새로운 인스턴스를 반환하며, 동일 아이디로 조회한 두 개의 인스턴스 레퍼런스 비교시 false가 된다.

> Main.java

```java
Crew crew = jpa.findCrew(Crew.class, id); //DB에서 조회
Crew crew2 = jpa.findCrew(Crew.class, id); //1차 캐시에서 가져옴

assertThat(crew).isEqualTo(crew2);
```

* 반면 JPA는 같은 트랜잭션일 때 같은 객체가 조회됨이 보장된다.

<br>

## 5. 마무리

ORM을 사용하면 개발 사이클을 다음과 같이 진행할 수 있다.

1. 도메인 설계
2. 비즈니스 로직 구현
3. 객체 테이블 매핑
4. 매핑 정보를 활용해 테이블 스키마 생성
5. DB 접근 로직 구현
6. 객체 속성 추가 및 테이블 컬럼 추가

DB 서버가 없는 상태에서도 개발 및 테스트를 진행할 수 있다. 아울러 DB에 의존적인 개발보다는, 도메인 및 비즈니스 로직을 중심으로 어플리케이션 개발에 집중할 수 있다.

장점은 다음과 같다.

* 패러다임 불일치 문제를 해결한다.
* 단순 반복적인 CRUD SQL 작성이 필요 없어 생산성이 향상된다.
* DB 벤더마다 미묘하게 다른 데이터 타입 및 SQL을 손쉽게 해결한다.
  * 데이터 접근을 추상화해두었다.
* 필드가 추가 및 삭제되는 경우 쿼리를 수정할 필요가 없으며, 엔티티만 수정하면 되는 등 유지보수가 쉽다.

단점으로는 높은 러닝 커브가 있겠다. 또한 복잡한 쿼리의 경우 JPQL 혹은 네이티브 쿼리를 사용해야 한다.

<br>

---

## References

* [[10분 테코톡] ⏰ 아마찌의 ORM vs SQL Mapper vs JDBC](https://www.youtube.com/watch?v=VTqqZSuSdOk)
