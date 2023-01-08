---
title:  "[Spring Framework 핵심 기술] 4장 : @Component Scan"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:09:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. @ComponentScan 주요 기능

* @ComponentScan은 Bean 스캔의 위치를 설정하고, 특정 애너테이션을 스캔할지 결정하는 필터 기능을 제공한다.
  * @Configurtation에 함께 위치시킨다.
* SpringBoot는 @SpringBootApplication 메타 애너테이션에 위 두 애너테이션이 포함되어 있어, 어플리케이션 실행 시 자동으로 Bean들이 등록된다.
  * 일반 Spring을 사용한다면 ``ApplicationContext context = new AnnotationConfigApplicationContext(ScanConfig.class)``를 통해 Bean들을 스캔하여 등록한다.

<br>

## 2. @Component

* @Repository.
* @Service.
* @Controller.
* @Configuration.

<br>

## 3. 동작 원리

* @ComponentScan은 스캔할 패키지와 Annotation에 대한 정보를 제공한다.
  * 실제 스캐닝은 ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에 의해 처리 된다.
* @Bean을 정의한 Configuration 클래스 파일 및 Function 등의 방식을 이용한 Bean보다 더 우선적으로 스캔되어 생성된다.


- @ComponentScan은 스캔의 위치를 설정하고, 특정 애너테이션을 스캔할지 결정하는 필터 기능을 제공한다.
    - @Configurtation에 함께 위치시킨다.
- @Repository, @Service 등은 @Component를 확장한 메타 애너테이션이다.
    - 원한다면 사용자가 커스텀 Component를 생성해 사용할 수 있다.
- 실제 스캐닝은 ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에 의해 처리 된다.
<br>

## 4. Function을 이용한 Bean 등록

> Application.java

```java
public static void main(String[] args) {
        new SpringApplicationBuilder()
        .sources(Demospring51Application.class)
        .initializers((ApplicationContextInitializer<GenericApplicationContext>)
  applicationContext -> {
                            applicationContext.registerBean(MyBean.class);
      })
      .run(args);
}
```

*	등록해야할 Bean이 많다면 @ComponentScan 방식은 처음 구동 시간이 오래 걸릴 수 있다.
*	Function을 이용하여 Bean을 등록하면 이러한 구동 시간 이슈를 해결할 수 있으며, 조금 더 유연하게 코드를 짤 수 있다.
*	설정이 복잡해진다는 단점이 존재한다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
