---
title:  "[Spring Framework 핵심 기술] 5장 : Bean Scope"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:10:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Bean Scope

* Singleton : 하나의 객체만 사용하는 사용하는 것으로서, 별도의 설정이 없을 때 기본값이다.
  * 초기 생성 이후 캐싱된 인스턴스를 사용하기 때문에 런타임 성능 측면에서 유리하다.
* Prototype : 매번 다른 객체를 사용하는 것을 의미하며, 상태가 있는 Bean을 사용할 때 쓴다.
  * 초기화 라이프사이클 콜백은 Bean Scope 범위에 상관없이 호출되지만, 소멸 라이프사이클 콜백은 프로토타입의 경우 호출되지 않는다.
  * IoC는 프로토타입 Bean 생성 뒤 클라이언트에게 건내는 순간 부터는 해당 인스턴스에 대한 기록을 중지한다.
    * 이후의 라이프 사이클(객체 메모리 해제, 객체가 쥐고 있는 리소스 반환)은 클라이언트가 제어해야 한다.
* Request, Session, Application, WebSocket : Bean 라이프사이클 범위 정의를 Web 개념에 맞추며, Wep 관련 ApplicationContext만 사용 가능하다.
* 좁은 라이프 사이클의 Scope의 Bean을 더 긴 라이프 사이클의 Bean에 주입할 때 AOP Proxy Bean을 주입해야 할 수도 있다.
  * Scope가 적용된 객체와 동일한 공용 인터페이스를 가진 프록시 객체를 사용한다.
    * 관련 Scope로부터 실제 타깃 객체를 되찾아와 메서드 호출을 위임할 수 있다.
* 필요하다면 커스텀 Scope를 정의할 수 있다.

<br>

## 2. Scope 예

> Single.java

```java
@Component
public class Single {

    @Autowired
    private Proto proto;

    public Proto getProto() {
        return proto;
    }
}
```

> Proto.java

```java
@Component @Scope("prototype")
```

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(ctx.getBean(Single.class).getProto());
        System.out.println(ctx.getBean(Single.class).getProto());
        System.out.println(ctx.getBean(Single.class).getProto());
    }
}
```

* Prototype이 Singleton을 참조하는 경우 문제가 없지만, Singleton이 Prototype을 참조하는 경우 프로퍼티인 Prototype Bean이 변경되지 않는 문제가 발생한다.
  * 위 예제는 Singleton Bean이 생성된 시점에 지닌 Proto의 동일한 주소 3개를 출력한다.
* 이러한 문제는 `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)` 와 같이 Proxy를 통해 해결할 수 있다.
  * scoped-porxy.
  * Object-Provider.
  * Provider(표준).

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
