---
title:  "[Spring Framework 핵심 기술] 9장 : ApplicationEventPublisher"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:14:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. ApplicationEventPublisher

* 옵저버 패턴의 구현체로서, 이벤트 프로그래밍에 필요한 인터페이스를 제공한다.
* ApplicationEventPublisher를 상속받는 ApplicationContext의 ``publishEvent(ApplicationEvent event)``를 사용한다.
* 이벤트를 발생시키는데 ApplicationEvent 클래스를 상속받았으나, Spring 4.2 부터는 상속받지 않아도 사용이 가능하다.

<br>

## 2. Spring 4.2 이전의 이벤트 발생 및 처리

> MyEvent.java

```java
public class MyEvent extends ApplicationEvent {
    //Bean이 아님
    private int data;

    public MyEvent(Object source, int data) {
        super(source);
        this.data = data;
    }

    public int getData() {
        return data;
    }
}
```

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationEventPublisher publishEvent;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        publishEvent.publishEvent(new MyEvent(this, 100)); //이벤트 발생
    }
}
```

> MyEventHandler.java

```java
@Component //Bean으로 등록
public class MyEventHandler implements ApplicationListener<MyEvent> {

    @Override
    public void onApplicationEvent(MyEvent myEvent) {
        System.out.println("event accpeted!" + myEvent.getData());
    }
}
```

* ApplicationListener를 구현한 클래스 만들어서 빈으로 등록한다.
* 이벤트가 발생하면 등록되어 있는 Bean 중 ApplicationListener를 구현한 클래스가 이를 처리한다.

<br>

## 3. Spring 4.2 이후의 이벤트 발생 및 처리

> MyEvent.java

```java
public class MyEvent {

    private Object source;
    private int data;

    public MyEvent(Object source, int data) {
        this.source = source;
        this.data = data;
    }

    public Object getSource() {
        return source;
    }

    public int getData() {
        return data;
    }
}
```

> MyEventHandler.java

```java
@Component
public class MyEventHandler {

    @EventListener
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public void onApplicationEvent(MyEvent myEvent) {
        System.out.println("event accpeted!" + myEvent.getData());
    }
}
```

* 특정 클래스를 상속받지 않고 @EventListener를 이용하여 간단하게 처리한다.
  * Framework의 코드가 나의 Event라는 코드에 침입하지 않아 유지보수 및 테스트가 용이해진다.
* 또 다른 EventListener가 있다면 기본적으로 Synchronized하게 실행된다.
* 순서를 설정하려면 @Order Annotation을 이용한다.
* 비동기적으로 설정하려면 EventListener에 @Async 및 Application 클래스에 @EnableAysnc Annotation을 각각 추가해준다.

<br>

## 4. Spring이 제공하는 기본 이벤트

* ContextRefreshedEvent : ApplicationContext를 초기화 및 리프레시 했을 때 발생한다.
* ContextStartedEvent : ApplicationContext를 start()하여 라이프사이클 Bean들이 시작 신호를 받은 시점에 발생한다.
* ContextStoppedEvent : ApplicationContext를 stop()하여 라이프사이클 Bean들이 정지 신호를 받은 시점에 발생한다.
* ContextClosedEvent : ApplicationContext를 close()하여 싱글톤 Bean이 소멸되는 시점에 발생한다.
* RequestHandledEvent : HTTP 요청을 처리했을 때 발생한다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
