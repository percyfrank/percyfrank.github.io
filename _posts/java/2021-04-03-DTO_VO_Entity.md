---
title: "DTO vs VO vs Entity"
excerpt: "DTO와 VO는 정확하게 어떤 차이가 있을까?"
categories:
  - Java
tags:
  - Java
toc: true
toc_sticky: true
last_modified_at: 2021-04-03T20:13:00-05:00
---

## 1. 혼란

개발을 진행하다가 구글링을 하면 용어가 뒤죽박죽이다. 누구는 VO라 하고, 누구는 DTO라고 하고. 정리된 글을 읽어도 두 개념 모두 큰 차이가 없고 비슷해 보인다. 정확하게 두 개념을 구분짓기 보다는 혼용하는 사람들이 더 많은 느낌이다. 공부 차원에서 개념들을 정리해보았다.

<br>

## 2. DTO(Data Transfer Object)

전송되는 데이터의 컨테이너이다. 데이터를 저장하여 사용하도록 하는 부분에서 필요하다. 특히 DTO는 Controller, Business Layer, Persistent Layer 등 계층 간의 데이터 교환을 위한 Java Beans를 의미한다.

VO와 비교하여 보면 DTO는 같은 시스템에서 사용되는 것이 아닌, 다른 시스템으로 전달하는 작업을 처리하는 객체이다. 개발 환경에서 보통 데이터는 다음과 같은 흐름으로 이동한다.

* 서버 측 : Database Column Data -> DTO -> API(JSON or XML) -> Client
* 클라이언트 측 : Server -> API(JSON or XML) -> DTO -> View or Local Database System

비즈니스 로직을 가지지 않는 순수한 데이터 객체이고, 가변의 성격을 갖고 있기 때문에 getter와 더불어 setter 메소드도 가진다.

<br>

## 3. VO(Value Object)

데이터 그 자체로 의미있는 것을 담고 있으며, 값 자체를 표현하는 객체다. 비즈니스 로직을 가질 수 있으며 Read-Only 및 불변성 특성상 getter만 가진다.

VO는 ``equals()`` 및 ``hashcode()``를 오버라이딩해야 한다. 두 VO 객체의 레퍼런스 주소가 달라도 내부에 선언된 값(필드)이 같으면 동일한 객체로 간주한다.

Enum이 대표적인 예이며, 클래스의 값을 중간에 바꿀 수 없기 때문에 새로 만들어야 한다는 특징이 있다. (Color.Red 등)

<br>

## 4. Entity

Entity 클래스는 DB의 테이블 내에 존재하는 컬럼만을 속성(필드)으로 가지는 클래스(데이터의 집합)이다. 다른 객체와 구별되는 식별자를 두는 것을 엔티티라고 한다. 엔티티 클래스는 상속을 받거나 인터페이스 구현체면 안 되며, 테이블 내에 존재하지 않는 컬럼을 가져서도 안 된다.

그렇다고 테이블과 완벽히 필드가 동일해야 엔티티인 것은 아니다. 특정 테이블과 매핑되고 식별자를 가진다면 엔티티로 볼 수 있다.

> SQL

```sql
create table if not exists LINE
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    color varchar(20) not null,
    primary key(id)
);
```

> Line.java

```java
public class Line { //엔티티임
    private Long id;
    private String name;
    private String color;
}

public class Line { //엔티티임
    private Long id;
    private String name;
}

public class Line { //엔티티 아님
    private Long id;
    private String name;
    private String color;
    private Sections sections;
}
```

* 위와 같은 테이블의 경우, 마지막 Line 클래스는 엔티티가 아니다.
  * Sections라는 다른 필드로 구성된 객체가 있기 때문이다.
* 엔티티 객체들의 동등성 비교를 위해 ``equals()``와 ``hashcode()``를 오버라이딩하는 경우, 식별자인 ID만 비교해도 충분하다.

> SQL

```sql
create table if not exists SECTION
(
    id bigint auto_increment not null,
    line_id bigint not null,
    up_station_id bigint not null,
    down_station_id bigint not null,
    distance int,
    primary key(id)
);
```

> Section.java

```java
public class Section {
    private Long id;
    private Long LineId;
    private Long upStationId;
    private Long downStationId;
    private Integer distance;
}
```

* 테이블이 기본형이라고 엔티티 클래스의 필드 타입을 이에 맞출 필요는 없다.
  * 필드로 ID를 가지면 DB에 의존적이며, Section이 필드로 직접 Station을 참조하는 것이 낫다.
  * OOP와 RDB 기술의 패러다임 불일치로 인한 문제이며, [ORM](https://xlffm3.github.io/spring%20data/orm-sqlmapper-jdbc/)을 통해 이러한 불일치를 해결할 수 있다.
* JPA를 사용할 때, ``@Transient`` 어노테이션을 사용하면 컬럼으로 매핑이 필요하지 않은 필드들을 가질 수 있다.

<br>

---

## References

* [Difference between DTO, VO, POJO, JavaBeans?](https://stackoverflow.com/questions/1612334/difference-between-dto-vo-pojo-javabeans)
* [[JAVA] DAO, DTO, VO 차이](https://lemontia.tistory.com/591)
* [VO vs DTO](https://ijbgo.tistory.com/9)
* [[정리] Entity(엔티티)](https://velog.io/@max9106/Entity)
