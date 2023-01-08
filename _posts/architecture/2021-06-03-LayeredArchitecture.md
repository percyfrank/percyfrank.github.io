---
title: "Layered Architecture"
excerpt: "확장성, 재사용성, 유지 보수 가능성, 가독성, 테스트 가능성이 우수하다."
categories:
  - Architecture
tags:
  - Architecture
date: 2021-06-03
last_modified_at: 2021-06-03
---

## 1. Layered Architecture란?

위키백과에서 정의한 Layered Architecture란 비즈니스 로직을 완전히 분리하여 데이터베이스 시스템과 클라이언트의 사이에 배치한 클라이언트 서버 시스템의 일종이다.

쉽게 말하자면 유사한 관심사들을 레이어로 나눠서 추상화하여 수직적으로 배열하는 아키텍처이다. 필요한 경우 한 층을 다른 것으로 교체해도 그 구조가 유지된다. 이 경우, 전체 시스템을 다른 환경에서도 사용할 수 있기 때문에 유연성 및 재사용성이 높다.

<br>

## 2. 3-Tier Architecture

3-Tier는 인프라(서버)의 구성 구조다.

### 2.1. Presentation Layer

프레젠테이션 계층은 응용 프로그램의 최상위에 위치하고 있으며, 서로 다른 층에 있는 데이터 등과 커뮤니케이션을 한다. GUI 및 Front-End의 영역이며, 사용자 인터페이스를 제공한다.

### 2.2. Application Layer

어플리케이션 계층은 비즈니스 로직 계층 또는 트랜잭션 계층이라고도 한다. 비즈니스 로직은 워크스테이션으로부터의 클라이언트 요청에 대해 마치 서버처럼 행동한다. 그것은 차례로 어떤 데이터가 필요한지를 결정하고, 메인프레임 컴퓨터 상에 위치하고 있을 세 번째 계층의 프로그램에 대해서는 마치 클라이언트처럼 행동한다. Middleware, 즉 Back-End의 영역이다.

### 2.3. Data Layer

데이터 계층은 데이터베이스와 그것에 액세스해서 읽거나 쓰는 것을 관리하는 프로그램을 포함한다. 애플리케이션의 조직은 이것보다 더욱 복잡해질 수 있지만, 3계층 관점은 대규모 프로그램에서 일부분에 관해 생각하기에 편리한 방법이다. Back-End 영역이지만 주로 DB 서버를 의미한다.

<br>

## 3. Spring MVC

계층형 아키텍쳐와 더불어 MVC를 짧게 정리해본다. 자세한 내용은 [DTO의 사용 범위에 대하여](https://xlffm3.github.io/spring%20&%20spring%20boot/DTOLayer/#11-mvc-%ED%8C%A8%ED%84%B4)를 참고하자. Spring은 MVC 패턴을 사용하여 웹 서비스를 제작한다. MVC 패턴이란 사용자 인터페이스와 비즈니스 로직을 분리하여 개발하는 방식을 말한다.

* 어플리케이션의 데이터에 해당되는, 비즈니스 규칙을 표현하는 도메인 Model.
* 모델을 사용자에게 보여주는, 프레젠테이션을 표현하는 View.
* 양측 사이에서 이를 제어하는 Controller.

### 3.1. Front Controller

MVC는 프론트 컨트롤러 패턴과 함께 사용된다. 서버로 들어오는 클라이언트의 요청을 가장 앞선에서 받아 처리하는 것이 프론트 컨트롤러이다. Spring은 DispatcherServlet이라는 프론트 컨트롤러를 제공하며, 해당 Servlet이 MVC 아키텍쳐를 관리한다.

DispatcherServlet의 작동 방식에 대한 글은 아니기 때문에 이에 대한 설명은 짧게 축약한다.

1. 프론트 컨트롤러가 요청을 받으면, HandlerMapping을 통해 요청을 처리할 수 있는 컨트롤러를 찾아 매핑한다.
2. HandlerAdapter를 통해 요청 처리를 위임하여 해당 컨트롤러가 요청을 처리한다.
3. 컨트롤러가 비즈니스 로직을 수행한 뒤 처리 결과 Model과 이를 출력할 View를 DispatcherServlet에게 반환한다.
4. 해당 정보를 바탕으로 DispatcherServlet은 ViewResolver를 통해 적합한 View 객체를 얻는다.
5. View 객체가 사용자에게 보여줄 응답 화면을 출력한다.

<br>

## 4. Layered Architecture 구현

![image](https://user-images.githubusercontent.com/56240505/95432828-1fe26680-098a-11eb-8a90-00f4d3342fc0.png)

![image](https://user-images.githubusercontent.com/56240505/95434101-d4c95300-098b-11eb-9d03-e6d3a586ebaa.png)

Layered Architecture는 유사한 관심사들을 레이어로 나눠서 추상화하여 수직적으로 배열하는 아키텍처다. MVC 패턴에서 Controller가 도메인 Model 객체들의 조합을 통해 프로그램의 작동 순서나 방식을 제어하는데, 어플리케이션의 규모가 커진다면 Controller는 중복되는 코드가 많아지고 비대해진다. 따라서 중복되는 코드들을 관심사에 맞춰 Service, DAO 등의 레이어로 분리시킨다.

* DDD의 Repository 패턴을 사용하는가, DAO를 사용하는가에 따라 용어 및 구분이 사뭇 다를 수 있다.
* 엄밀하게 말하면 Repository는 Domain 단이지만, 이 글에서는 Repository와 DAO를 비슷한 개념으로 보고 설명한다.

### 4.1. 장점

> 시스템을 레이어로 나누면 시스템 전체를 수정하지 않고도 특정 레이어를 수정 및 개선할 수 있어 재사용성과 유지보수에 유리하다.

1. 단방향 의존성.
  * 각각의 Layer는 하위의 인접한 Layer에게만 의존한다.
2. 관심사의 분리.
  * 각 레이어는 주어진 고유한 역할만을 수행한다.
  * 표현 계층에는 비즈니스 로직이 없으며, 비즈니스 계층에는 DB 관련 로직이 존재하지 않는다.
3. 확장성과 재사용성 및 유지 보수 가능성.
  * 각 Layer가 서로 독립적이고 역할이 분명하기 때문에, 서로에게 끼치는 영향을 최소화하며 확장 및 수정이 가능하다.
  * Layer가 독립적이면, 하나의 Service Layer는 여러 개의 Presentation Layer가 재사용할 수 있다.

요약하자면 다음과 같다.

* 역할이 명확하게 분리되어 있어 코드의 가독성도 높다.
* 범위가 명확하고 확실하여 기능을 테스트하기 쉽다.
* **확장성, 재사용성, 유지 보수 가능성, 가독성, 테스트 가능성** 이 우수하다.

<br>

## 5. 구성

### 5.1. Presentation Layer

> **Presentation Layer == Web Layer == UI Layer**

화면 조작 또는 사용자의 입력을 처리하기 위한 관심사를 모아놓은 레이어다.

* 최종 사용자에게 UI를 제공하거나 클라이언트로 응답을 다시 보내는 역할을 담당하는 모든 클래스다.
* 즉, API의 엔드포인트들을 정의하고 전송된 HTTP 요청들을 읽어 들이는 로직을 구현한다.

### 5.2. Application Layer

> **Application Layer == Service Layer**

일반적으로 Domain 모델의 비즈니스 로직 1개를 호출해서는 복잡한 요청을 처리할 수 없다. 다양한 Domain과 비즈니스 로직을 사용하여 응답을 처리하는 Layer가 필요하다. **여러 비즈니스 로직을 의미있는 수준으로 묶어 제공하는 역할을 한다.**

단, 핵심 비즈니스 로직을 직접 구현하지 않고 얇게 유지한다.

* 객체지향 프로그램은 고유한 책임을 수행하는 협력을 통해 작동한다.
  * 따라서 핵심이 되는 비즈니스 로직은 Service Layer가 아닌 Domain 모델들이 가져야 한다.
  * 각각의 Domain 모델은 할당된 책임을 수행하며 협력해야 한다.
* 여러 Domain들을 생성, 조합, 협력시키는 어플리케이션(응용) 로직을 순수한 Domain 객체에 넣지 않는다.
  * 과한 의존성이 생겨 테스트 하기 어렵고, 결합도가 높아지며 응집도가 떨어진다.

Service Layer가 Domain 모델 및 Presentation Layer 사이에서 Domain 기능들을 응용하는 로직을 관장한다. Service Layer는 트랜잭션 및 Domain 모델의 기능(비즈니스 로직) 간의 순서만을 보장해야 한다. **핵심 비즈니스 로직은 Domain 모델의 책임임을 명심하자.** **Domain 모델을 캡슐화하는 역할이다.**

* 핵심 비즈니스 로직을 가진 Domain 모델을 View, Controller 등 표현 계층에 노출시키는 것은 [보안 등 여러 단점이 존재한다.](https://xlffm3.github.io/spring%20&%20spring%20boot/DTOLayer/#12-%EC%9A%A9%EB%A1%80)
* Service Layer가 표현 계층에게 직접적인 Domain 모델을 반환하지 않고, DTO 등을 반환하면 표현 계층과 Domain 모델간의 강결합 의존성을 끊을 수 있다.
  * 즉, Service Layer는 Domain 모델을 캡슐화하여 보호하는 역할도 겸한다.
  * 단일 책임 원칙 및 유지보수가 수월해진다.

### 5.3. Domain Layer

> **Domain Model == Business Layer**

Data와 그와 관련된 비즈니스 로직(핵심 기능)을 가진 객체이다. 무조건 DB와 관계가 있는 Entity만을 의미하는 것은 아니며, VO처럼 값 객체들도 이 영역이다.

* Domain 객체 모델과 DB 스키마 사이의 불일치가 발생할 수 있다.
  * Mapper를 중간에 둬서 도메인 쪽에 있는 객체와 DB 간의 결합도를 끊어 준다.
    * 즉, 도메인 객체는 DB에 대해 독립적이어야 한다.
* 데이터 매퍼는 흔히 ``ORM(Object Relational Model)``이라고 한다.

후술할 Repository는 엔티티 객체를 보관하고 관리하는 컬렉션으로, 엄밀하게 말하면 Domain 레이어에 속한다. Repository의 구현체가 Infrastructure 레이어에 속한다.

### 5.4. Infrastructure Layer

> **Infrastructure Layer == Persistence Layer**

RDBMS 뿐만 아니라 Message Queue 및 HTTP 통신과 같은 여러 구현 기술이 위치한 레이어다. 흔히 말하는 DAO 및 Repository의 구현체가 이에 속한다.

* Data Source를 추상화한다.
  * DB에서 데이터를 저장, 수정, 불러 들이는 등 DB와 관련된 로직을 구현한다.
  * DAO는 데이터에 접근하도록 DB접근 관련 로직을 모아둔 객체다.
* 표현, 응용, 도메인을 영역을 지원하는 역할을 한다.

<br>

---

## References

* [3 Tier Architecture(3계층 구조)](http://blog.naver.com/PostView.nhn?blogId=limoremo&logNo=220073573980)
* [[Spring] Spring MVC와 Dispatcherservlet](https://gangnam-americano.tistory.com/59)
* [[Spring] MVC : Controller와 Service의 책임 나누기](https://umbum.dev/1066)
* [Spring Layered Architecture](https://yoonho-devlog.tistory.com/25)
* [[Layer Architecture] 레이어 아키텍처](https://black-jin0427.tistory.com/m/207)
* [Layered Architecture](https://velog.io/@soojung61/Layered-Architecture)
* 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (이동욱 저)
