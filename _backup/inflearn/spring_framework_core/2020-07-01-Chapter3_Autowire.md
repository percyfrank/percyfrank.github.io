---
title:  "[Spring Framework 핵심 기술] 3장 : @Autowired"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:08:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. @Autowired

* 필요한 의존 객체의 "타입"에 해당하는 빈을 찾아 주입한다.
* 해당 Annotation이 붙어있으면 의존성 주입을 시도하며, 실패하면 어플리케이션 구동이 실패한다.
* 따라서 의존성 주입이 Optional인 경우, ``@Autowired(required = false)`` 옵션을 줄 수 있으며 기본값은 true이다.
* 생성자와 세터 및 필드에 사용할 수 있다.
  * 해당 애너테이션을 사용하는 생성자가 여러 개인 경우, 의존성 주입이 가장 많이 만족되는 생성자를 사용한다.

<br>

## 2. 같은 타입의 Bean이 여러 개인 경우

* BookRepository라는 인터페이스를 필드로 가지고 있을 때, 해당 인터페이스를 구현하는 여러 가지 Bean들 중 원하는 Bean을 찾기 힘들다.
* 사용하고자 하는 Bean의 @Component 관련 Annotation 옆에 @Primary를 추가한다.
* 혹은 @Autowired Annotation 옆에 @Qualifier("Bean Name")를 추가한다.
* 혹은 List를 통해 해당 타입의 Bean을 모두 주입받는다.

<br>

## 3. Lifecycle

* Bean의 라이프사이클 콜백 작업을 수행할 수 있다.
  * InitializingBean, DisposableBean 인터페이스를 구현하여 콜백 작업을 수행할 수 있으나 프레임워크 코드와의 결합도가 증가하기 때문에 권장하지 않는다.
  * 커스텀하게 ``init()``과 ``destroy()`` 메서드를 정의하고 XML 파일 등에 등록할 수 있으나, AOP Proxy 인터셉터 등과 함께 사용할 때 결과가 일정하지 않을 수 있다.
    * 가장 모던한 방법은 @PostConstruct, @PostDestroy 애너테이션을 사용한다.
* ``BeanPostProcessor`` : 새로 만든 Bean 인스턴스를 수정할 수 있는 라이프 사이클 인터페이스다.
  * 효력 범위는 선언된 컨테이너 내부로 한정된다.
* ``AutowiredAnnotationBeanPostProcessor`` : Spring이 제공하는 @Autowired와 @Value 및 JSR-330의 @Inject Annotation을 지원하는 처리기이다.
  * Bean Initialization 이전에, AutowiredAnnotationBeanPostProcessor 클래스가 동작하여 @Autowired의 작업을 수행하여 주입한다.
  * 이후 @PostConstruct 같은 콜백 애너테이션이 동작한다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
