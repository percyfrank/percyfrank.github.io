---
title: "DDD 및 Aggregate"
excerpt: "도메인 주도 설계에 대해 알아보자."
categories:
  - DDD
tags:
  - DDD
date: 2021-06-10
last_modified_at: 2021-06-10
---

## 1. Domain

* Domain의 사전적인 의미는 '정보와 활동의 영역'이다.
* 프로그래밍 관점에서는 Domain은 어플리케이션 내 로직들이 관여하는 정보와 활등의 영역이다.
* 어떤 서비스의 회원가입 및 회원탈퇴 등의 작업은 모두 회원과 관련되어 있으며, 회원이 Domain이다.

<br>

## 2. Domain Model

* 특정 Domain을 개념적으로 표현한 것이다.
* 지식을 공유하고 소통하는 도구이다.
* 개발자는 구축한 Domain Model을 통해 요구사항을 제대로 이해했는지 확인할 수 있다.
* 기획자는 의견을 맞출 때 Domain Model을 사용해서 소통할 수 있다.
* [Layered Architecture](https://xlffm3.github.io/spring%20&%20spring%20boot/LayeredArchitecture/) 상의 Domain Layer의 결과물을 Domain Model이라고 한다.
  * Domain Layer는 Domain의 개념과 정보 및 규칙 등을 표현하는 책임을 진다.

<br>

## 3. DDD(Domain Driven Design)

* DDD는 도메인을 중심으로 설계하는 디자인 방법론이다.
* Domain은 실세계에서 사건이 발생하는 집합이다.
  * 여러가지 Domain들이 서로 상호 작용하도록 어플리케이션을 설계한다.
* 객체지향을 이룩하는데 도움이 되는 방법론이다.
  * 객체지향이란 자율적인 책임과 역할을 가진 객체들이 공동의 목표(어플리케이션 정상 구동)을 위해 서로 협력하는 것이다.
  * DDD를 통해 어떤 객체가 필요하며, 어떻게 추출할 것이며, 어떻게 협력할 것인지 등을 고민할 수 있다.

### 3.1. 필요성

* 개발을 진행하다 보면 객체를 단순히 데이터를 전달하는 용도로서 사용하는 경우가 많다.
  * 대표적으로 객체의 모델링이 DB 테이블 모델링에 종속적이게 되는 등 객체가 테이블에 맞추어 데이터를 전달하는 역할로 전락한다.
  * RDB는 어떻게 데이터를 저장할지에 초점을 맞춘 반면, OOP는 기능과 속성을 한 곳에 관리하기 때문에 패러다임 불일치가 발생한다.
* Domain 모델링과 실제 개발이 불일치하는 것을 해결하는 것이 DDD다.
  * DDD를 적용하면 기획자와 개발자 및 디자이너 등 비개발 팀원들도 똑같이 유비쿼터스 언어로 소통할 수 있다.
  * 유비쿼터스 언어란 Domain 언어를 이해관계자들이 공통적으로 이해할 수 있도록 정의하는 것이다.
* 따라서 주요 비즈니스 로직은 응용 계층이 아닌 Domain Model이 가진다.

<br>

## 4. DDD의 기본 요소

### 4.1. Entity

* Entity는 다른 Entity와 구별될 수 있는 고유한 식별자를 가진다.  

### 4.2. Value

* Value는 불변을 원칙으로 한다.
* 의미를 명확하게 표현하거나 두 개 이상의 데이터가 개념적으로 하나인 경우 사용한다.
* 식별자가 존재하지 않는다.

### 4.3. Aggregate

![image](https://user-images.githubusercontent.com/56240505/121492967-51786980-ca12-11eb-9d91-702f6ae9d84a.png)

* 데이터 변경의 단위로 다루는 연관 Entity 객체의 묶음(집합)을 의미한다.
* Aggregate에 속한 Entity 객체 중 Aggregate 집합에 속한 객체들을 관리하는 Entity를 Aggregate Root라고 한다.
* Order는 여러 Entity들과 연관 관계를 맺기에, Order를 Aggregate Root로 하는 Aggregate가 형성되었다.
* Aggregate에 속한 객체는 다른 Aggregate에 속하지 않는다.
* 경계 안의 객체는 서로를 참조할 수 있으나 경계 밖의 객체는 Aggregate 구성 요소 중 Aggregate Root에만 접근할 수 있다.
* 각 Aggregate는 자기 자신을 관리할 뿐, 다른 Aggregate는 관리하지 않는다.
* Aggregate로 묶어서 바라보면 상위 수준에서 Domain Model 간의 관계를 파악할 수 있다.

### 4.4. Repository

* Aggregate 단위로 Domain 객체를 저장하고 조회하는 기능을 정의한다.
* Repository 메소드는 완전한 Aggregate를 제공해야 한다.
  * Repository가 완전한 Aggregate를 제공하지 않으면 필드에 null이 들어갈 것이고, 이는 NPE를 유발한다.
* 모든 Entity 저장 및 접근은 Repository에 위임하고, 클라이언트는 Domain Model에 집중하도록 한다.
* DAO를 사용하는 경우 테이블 별로 DAO를 작성했다면, Repository는 클라이언트가 실질적으로 접근하는 Aggregate Root에 대해서만 제공한다.
  * DDD에서는 전역 식별성을 가진 Entity가 Aggregate Root다.
    * 배송지와 지불 방식 및 상품 등과 관련된 상호 작용을 보면 Order과 Root임을 직관적으로 파악할 수 있다.
  * Order가 아닌 Payment 등에만 데이터 쓰기 작업을 수행하면 데이터 무결성이 깨진다.
  * 따라서, Aggregate 단위로 쓰기 작업을 진행해야 한다.

<br>

---

## References

* [DDD, Aggregate Root 란?](https://eocoding.tistory.com/36)
* [DDD](https://kchanguk.tistory.com/139)
