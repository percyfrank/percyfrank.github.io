---
title:  "[Spring Framework 핵심 기술] 17장 : Proxy 기반 AOP"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:22:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Spring AOP

* Proxy 기반의 AOP 구현체이며, Bean에만 AOP를 적용할 수 있다.
* 모든 AOP 기능을 제공하는 것이 목적이 아니라, Spring IoC와 연동하여 엔터프라이즈 어플리케이션에서 가장 흔한 문제에 대한 해결책을 제공하는 것이 목적이다.
* 인터페이스 기반의 JDK 다이내믹 프록시를 사용하지만, 비즈니스 오브젝트가 인터페이스를 구현하지 않으면 기본적으로 클래스 기반의 CGLIB 프록시를 사용한다.

<br>

## 2. Proxy Pattern

![Proxy Pattern](https://user-images.githubusercontent.com/56240505/80108193-fdfd4f00-85b6-11ea-863a-c54984ba5e15.png)<br><br>

> SimpleEventService.java

```java
@Service
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {
        System.out.println("CREATE");
    }

    @Override
    public void publishEvent() {
        System.out.println("PUBLISH");
    }
}
```

> ProxyEventService.java

```java
@Service
@Primary
public class ProxyEventService implements EventService {

    @Autowired
    SimpleEventService simpleEventService;

    @Override
    public void createEvent() {
        System.out.println("시작 시간 계산");
        simpleEventService.createEvent();
        System.out.println("시작 시간 계산 완료");
    }

    @Override
    public void publishEvent() {
        System.out.println("시작 시간 계산");
        simpleEventService.publishEvent();
        System.out.println("시작 시간 계산 완료");
    }
}
```

> AppRunner.java

```java
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.publishEvent();
    }
}
```

* 프록시 패턴이란 프록시 객체가 진짜 객체를 감싸서 클라이언트의 요청을 처리하게 하는 패턴이다.
* 클라이언트는 인터페이스로 Proxy 객체를 사용하게 된다.
  * Proxy도 Subject를 구현한 다음 Primary 어노테이션을 지정하여 우선 사용하게 한다.
* 해당 Proxy는 내부에서 동일 메서드를 RealSubject의 것을 delegate하고, 추가 작업을 진행할 수 있다.
  * 시간 측정, 로깅, 로그인 유효성 검사 등등.
* 기존 코드 변경 없이 접근 제어 또는 부가 기능 등을 추가할 수 있다.

<br>

## 3. 문제점

* 프록시 클래스 작성 비용 및 객체 관리 등 단점이 존재한다.
  * 결국 코드의 중복이 발생하게 되며, 효율적이지 못하다.
  * Spring IoC 컨테이너가 제공하는 기반 시설과 Dynamic 프록시를 사용하여 여러 이러한 이슈를 해결한다.
* 동적 프록시 : 동적으로 프록시 객체 생성하는 방법이며, Java가 제공하는 방법은 인터페이스 기반의 프록시 생성이다.
  * CGlib은 클래스 기반 프록시도 지원한다.
* Spring IoC : 런타임시 기존 Bean을 대체하는 프록시 Bean을 동적으로 생성해 등록하며, 클라이언트 코드 변경이 없다.
  * ``AbstractAutoProxyCreator implements BeanPostProcessor``
  * BeanPostProcessor는 Lifeycycle 인터페이스 중 하나로 새로 생성된 Bean을 조작하는 기능이 있다.
* AbstractAutoProxyCreator는 특정 Bean을 감싸는 프록시 Bean을 만들고, 해당 프록시 Bean을 실제 Bean 대신에 주입시켜준다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
