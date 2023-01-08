---
title:  "[Spring Framework 핵심 기술] 18장 : @AOP"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:23:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Dependency 추가

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

<br>

## 2. Aspect와 Advice 및 Pointcut 정의

> PerfAspect.java

```java
@Aspect
@Component
public class PerfAspect {

    //@Around("execution(* com.example..*.EventService.*(..))")
    //@Around("@annotation(PerLogging)")
    @Around("bean(SimpleEventService)")
    public Object logPeer(ProceedingJoinPoint pip) throws Throwable {
        long begin = System.currentTimeMillis();
        System.out.println();
        Object retval = pip.proceed(); //실제 원하는 Target Method의 작업 수행
        System.out.println(System.currentTimeMillis() - begin);
        return retval;
    }

    @Before("bean(SimpleEventService)")
    public void hello() {
        System.out.println("hello");
    }
}
```

> PerLogging.java

```java
@Documented
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.METHOD)
public @interface PerLogging {
}
```

* @Aspect를 정의하며, Bean 등록을 위해 컴포넌트 스캔을 사용한다면 @Component도 추가한다.
* 위 메서드들 각각은 Advice며, 각 Advice의 Pointcut 정의에 사용되는 어노테이션은 다음과 같다.
  * @Before
  * @AfterReturning
  * @AfterThrowing
  * @Around
  * @Pointcut(표현식)
* 주요 표현식.
  * ``execution.``
  * ``@annotation``
  * ``bean``
* Pointcut 조합.
  * &&
  * ||
  * !
* 특정 Annotation 지시를 통해 일부 메서드의 AOP 지정을 누락시킬 수 있다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
