---
title: "Foreign Key의 Cascade 옵션"
excerpt: "부모 테이블과 자식 테이블간의 제약과 종속성에 대해 알아보자."
categories:
  - Database
tags:
  - Database
date: 2021-06-10
last_modified_at: 2021-06-10
---

## 1. Foreign Key

> SQL

```sql
create table if not exists STATION
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    primary key(id)
);

create table if not exists SECTION
(
    id bigint auto_increment not null,
    line_id bigint not null,
    up_station_id bigint not null,
    down_station_id bigint not null,
    distance int not null,
    primary key(id),
    foreign key (up_station_id) references station(id),
    foreign key (down_station_id) references station(id)
);
```

* Section 테이블은 외래키로 Station 테이블의 ID를 참조하고 있다.
* 이처럼 FK를 지정해두면, Station 테이블에 존재하지 않는 ID 값이 Section 테이블의 station_id 관련 칼럼에 들어갈 수 없다.
* 즉, 참조 무결성을 지킬 수 있다.

> SQL

```sql
Error Code: 1451. Cannot delete or update a parent row: a foreign key constraint fails
```

* Section이 참조하고 있는 부모 테이블 Station의 ID는 데이터 정합성을 위해 수정이나 삭제가 불가능하다.
* 반면 참조되고 있지 않는 ID는 자유롭게 수정이나 삭제가 가능하다.
* 별도의 옵션을 주지 않더라도 FK 제약 조건 설정시 아래의 제약 조건들도 함께 설정되기 때문이다.
  * ``ON DELETE RESTRICT`` : 삭제시 제약.
  * ``ON UPDATE RESTRICT`` : 갱신시 제약.

<br>

## 2. Cascade

Cascade의 사전적인 의미는 종속이다. 부모 테이블의 값이 수정이나 삭제가 발생하면, 해당 값을 참조하고 있는 자식 테이블의 역시 종속적으로 수정 및 삭제가 일어나도록 하는 옵션이다.

> SQL

```sql
create table parent
(
    id bigint not null,
    name varchar(255) not null,
    primary key(id)
);

create table child
(
    id bigint not null,
    name varchar(255) not null,
    parent_id bigint not null,
    primary key(id),
    foreign key(parent_id) references parent(id) on delete cascade on update cascade
);
```

* ``on delete cascade``와 ``on update cascade``를 사용하면 참조되고 있는 부모 테이블의 값을 수정 및 삭제시, 해당 값을 참조하고 있는 자식 테이블의 값이 함께 수정 및 삭제된다.

<br>

---

## References

* [CONSTRAINT 와 CASCADE 의 활용](https://blog.ycpark.net/entry/FOREIGN-KEY-%EC%99%80-CONSTRAINT-%EC%9D%98-%EC%82%AC%EC%9A%A9)
