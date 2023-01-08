---
title:  "[Spring Framework 핵심 기술] 6장 : Profile"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:11:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Environment

* Environment 인터페이스는 Profile과 Property를 다룬다.
* Profile은 Bean들의 집합이며,  Environment의 역할은 활성화 할 Profile의 확인 및 설정이다.
* 특정 환경에서는 사용하고 싶은 Bean이 다를 경우 이용한다.
  * 테스트 환경 구성 등이 대표적인 유스케이스이다.
* 클래스 및 메소드에 각각 정의할 수 있다.
  * 클래스에 정의하는 경우 Configuration 클래스에 정의하거나, 해당 Bean에 직접 정의한다.
  * 메소드에 정의하는 경우 `@Bean @Profile("test")` 를 사용한다.
* 설정 방법.
  * `Dspring.profiles.avtive=”test,A,B,..."` 혹은 @ActiveProfiles Annotation을 통해 Profile을 설정한다.
  * Spring Boot의 경우 application.propertis에 spring.profiles.active=TEST 등의 설정이 가능하다.

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Environment evn = ctx.getEnvironment();
        System.out.println(Arrays.toString(environment.getActiveProfiles())); //[]
        System.out.println(Arrays.toString(environment.getDefaultProfiles())); //[default]
    }
}
```

* EnvironmentCapable을 상속받는 ApplicationContext의 ``getEnvironment()`` 메소드를 사용한다.
* 별도의 Profile을 지정해주지 않는다면, Bean들은 기본적으로 Default Profile에 속한다.

<br>

## 2. Profile 실습

> TestConfiguration.java

```java
@Configuration @Profile("test")

public class TestConfiguration {

  @Bean //여기에 @Profile("test")도 가능함
  public BookRepository bookRepository() {
      return new BookRepository();
  }
}
```

> TestBookRepository.java

```java
@Repository @Profile("test")
```

* 특정 환경에서는 사용하고 싶은 Bean이 다를 경우 이용한다.
  * 어플리케이션이 test라는 프로파일로 실행되지 않는 이상 해당 configuration에 정의된 Bean을 사용할 수 없다.
* 설정 방법.
  * ``-Dspring.profiles.avtive=”test,A,B,..."`` 혹은 @ActiveProfiles Annotation을 통해 Profile을 설정한다.
  * Spring Boot의 경우 application.propertis에 spring.profiles.active=TEST 등의 설정이 가능하다.

<br>

## 3. Profile 표현식

*	! (not).
*	& (and).
*	| (or).

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
