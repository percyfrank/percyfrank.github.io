---
title:  "[Spring Framework 핵심 기술] 1장 : IoC & Bean"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:06:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. IoC(Inversion of Control)

* 의존 관계 주입(Dependency Injection)을 뜻한다.
* 어떤 객체가 사용하는 의존 객체를 직접 만들어서 사용하는 것이 아니라, 주입 받아 사용하는 방식을 의미한다.
* 객체의 생성 위치와 시점 및 생명 주기 등의 제어권을 객체 스스로가 통제하는 않는다.
  * 외부의 다른 오브젝트(IoC Container)가 제어권을 가지고 있다.
  * 객체의 생성 책임과 사용 책임을 분리한다.

<br>

## 2. Spring IoC Container

* Spring IoC Contianer란 IoC 기능 및 Bean을 담고 있는 Container다.
  * Bean이란 IoC Container가 의존 관계 설정(주입) 및 생성 등 관리하여 제공하는 객체를 의미한다.
  * Container는 Bean 설정 소스로부터 Bean 정의를 읽어들인 뒤, Bean을 구성하여 제공한다.
* 어플리케이션 컴포넌트의 중앙 저장소이며, 최상위 인터페이스는 `BeanFactory`다.
* ``ApplicationContext`` : ``BeanFactory``를 확장한 인터페이스이다.
  * 메시지 소스 처리 기능(i18n).
  * 이벤트 발행 기능.
  * 리소스 로딩 기능.
  * WebApplicationContext와 같이 응용 계층 특화 컨텍스트.
  * Spring AOP 기능과의 쉬운 통합.

<br>

## 3. Bean

* Bean이란 IoC Container가 의존 관계 설정(주입) 및 생성 등 관리하여 제공하는 객체를 의미한다.
  * IoC Container에 제공하는 설정 메타파일을 기반으로 구성된다.
* Bean과 관련한 명세는 BeanDefinitions 객체에 정의(맵핑)된다.
  * 클래스, 이름, 스코프, 생성자, 프로퍼티, 라이프사이클 콜백, 의존성 등.
* Container 외부에서 유저에 의해 생성되어 존재하는 객체를 Bean으로 등록할 수 있다.
  * ApplicationContext의 ``getBeanFactory()``를 통해 ``DefaultListableBeanFactory``를 구현한 BeanFactory을 얻어 등록이 가능하다.
    * 그러나 런타임에 수동으로 Bean을 등록하는 것은 권장하지 않는다.
    * 등록하더라도 의존성 주입 및 동시성 문제 등으로 인해 가능한 빨리 등록해야 한다.
* 기본 생성자와 Getter-Setter를 가진 JavaBean 이외의 객체들도 Bean으로 등록이 가능하다.
* 특정 Bean의 실제 런타임 타입은 Bean 설정 메타파일에 명시된 클래스와 다를 수 있다.
  * ``BeanFactory.getType()`` 으로 특정 이름의 Bean의 런타임 타입을 알 수 있다.

<br>

## 4. 의존성

* 의존성 주입이란 생성자, 팩토리 메서드, Setter 등의 인자를 통해 의존성을 정의하는 것을 뜻한다.
* IoC 컨테이너는 Bean을 생성할 때 의존성을 주입한다.
  * 객체 스스로가 의존 관계 객체를 선택하고, 원하는 위치 및 시점에서 인스턴스화되던 기존의 제어 흐름과 반대된다.
  * 이를 통해 결합도가 하락한다.
  * 의존 Bean이 추상 클래스나 인터페이스인 경우, Mock을 통해 테스트하기 쉬워진다.
* [필드 주입 vs 생성자 주입 vs Setter 주입]([https://xlffm3.github.io/spring & spring boot/DI/](https://xlffm3.github.io/spring%20&%20spring%20boot/DI/))
  * 생성자 주입을 사용하는 경우 Bean들간의 순환 의존성 문제가 발생할 수 있다.
    * Spring IoC Container는 로드되는 시점에 순환 의존성 등 설정 파일 문제를 감지한다.
    * 비용이 더 들지라도 순환 의존성 등 설정 파일 문제를 나중이 아닌 ApplicationContext 로드 시점에 인지하고자, 기본적으로 ApplicaitonContext 구현체들은 Singleton Bean들을 미리 인스턴스화한다.
    * 일반적인 Scope의 Bean은 요청이 발생할 때 값이 설정되고 의존성이 해결된다.
* DI를 사용하면 단위 테스트를 수행하기 쉬워진다.
  * IoC 컨테이너 없이 직접 new 연산자로 객체를 생성할 수도 있다.
  * Mock 테스트를 수행하기 수월하다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
