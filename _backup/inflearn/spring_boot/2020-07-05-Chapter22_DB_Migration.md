---
title:  "[Spring Boot 개념과 활용] 22장 : DB 마이그레이션"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-13T08:12:00-05:00
---

## 1. DB Migartion

* 일종의 DB 버전 관리 기능이다.
* Spring Boot에서 대표적으로 사용할 수 있는 툴은 Flyway와 Liquibase가 있다.
* 의존성 : ``org.flywaydb:flyway-core``

<br>

## 2. Flyway

* resources 밑 db/migration 또는 db/migration/{vendor} 디렉토리를 경로로 지정한다.
  * spring.flyway.locations로 변경할 수 있다.
* V숫자__이름.sql
  * V는 꼭 대문자로 써야한다.
* 숫자는 순차적으로 타임스탬프를 권장한다.
* 숫자와 이름 사이에 언더바는 두 개이다.
* 이름은 가능한 서술적으로 작성한다.

<br>

## 3. 사용 예제

* ``spring.jpa.hinernate.ddl-auto=validate``로 설정하고, ``spring.jpa.generate-ddl=false``로 설정한다.
* JPA 초기화 기능이 꺼진 상태에서, shcema.sql 등 초기화 sql을 삭제한다.
  * Flyway가 Schema를 초기화하는 sql 파일이 없다면 validate로 에러가 발생한다.
  * V1__init.sql 등으로 Schema를 초기화하는 Flyway 파일을 추가하면 예외가 발생하지 않는다.
* 새로운 컬럼을 추가한 경우, Flyway에서는 Schema 변경이 없기 때문에 validate에서 맵핑 에러가 발생한다.
  * 기존 V1은 절대 건들이지 않고, V2 등 다음 Flyway 파일을 추가하여 새로운 칼럼을 Alter한다.
  * Flyway에서 Shcema 변경 뿐만 아니라 데이터 조작이 가능하다.
  * IDE 상에서 DB의 Flyway를 참조하면 버전 정보를 확인할 수 있다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
