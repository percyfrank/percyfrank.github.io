---
title:  "[Spring Boot 개념과 활용] 5장 : SpringApplication"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:07:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. SpringApplication

> DemoApplication.java

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplicationBuilder().sources(Application.class).build();
        springApplication.run(args);
        // SpringApplication.run(DemoApplication.class, args);
    }
}
```

* Static하게 클래스를 참조하여 실행할 수도 있으며, 세부적인 컨트롤이 필요하다면 객체를 생성하여 실행할 수 있다.
* IDE 상의 VM Option에 -Ddebug나, 프로그램 실행 시 --debug를 넣어주면 디버그 모드가 실행된다.
* FailureAnalyzer.
  * 에러에 대한 정보 출력을 커스텀하게 조정할 수 있다.
* Banner
  * banner.txt | gif | jpg | png 등을 resources(classpath) 디렉토리에 생성한다.
  * 또는 spring.banner.location으로 위치를 지정할 수 있다.
  * ${spring-boot.version} 등의 변수를 사용할 수 있다.
  * Banner 클래스 구현하고 ``SpringApplication.setBanner()``로 설정하거나 끌 수 있다.
* 빌더 패턴이 가능하다.

<br>

## 2. ApplicationEvent 등록

> SampleListener.java

```java
//ApplicationContext 생성 이전의 이벤트라 @Component 등록이 필요없다.
public class SampleListener implements ApplicationListener<ApplicationStartedEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("hi started!");
    }
}
```

> DemoApplication.java

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(DemoApplication.class);
        springApplication.addListeners(new SampleListener());
        springApplication.run(args);
        // SpringApplication.run(DemoApplication.class, args);
    }
}
```

* Listener를 통해 이벤트를 등록할 수 있는데, ApplicationContext가 발생하는 시점에 따라 main 메서드에서의 사용법이 상이하다.
  * 레퍼런스에서 제공되는 ApplicationStartingEvent의 경우 ApplicationContext가 생성되기 이전의 이벤트이다.
  * 따라서 해당 EventListener를 Bean으로 등록해도 이벤트가 Catch되지 못한다.
  * 위 예제 코드처럼 그러한 예외적인 시점의 이벤트는 springApplication에 리스너를 추가해야한다.
* 일반적인 Event의 경우 ApplicationContext 생성 이후 발생하기 때문에, 별도의 리스너 추가 없이 EventListener를 Bean으로 등록하여 사용이 가능하다.

<br>

## 3. WebApplicationType 설정

> DemoApplication.java

```java
springApplication.setWebApplicationType(WebApplicationType.NONE);
```

* SpringApplication은 적합한 타입의 ApplicationContext를 생성한다.
  * Spring MVC가 존재하면 AnnotationConfigServletWebServerApplicationContext를 사용한다.
  * Spring MVC가 존재하지 않고, Spring WebFlux만 존재하면 AnnotationConfigReactiveWebServerApplicationContext를 사용한다.
  * 그 외, AnnotationConfigApplicationContext를 사용한다.

<br>

## 4. ApplicationArguments 사용

> AppRunner.java

```java
@Component
@Order(1)
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(args.containsOption("foo"));
    }
}
```

* 파라미터로 받는 ApplicationArguments는 Bean으로 자동 등록된다.
* 프로그램 인자로 --XX를 주며, IDE 상의 VM 옵션은 인식하지 못한다.

### 4.1. 부수 작업

> CommandRunner.java

```java
@Component
@Order(2)
public class CommandRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("command line runner!");
    }
}
```

* ApplicationRunner(추천) 또는 CommandLineRunner를 구현한 클래스의 run 메소드를 통해 부수적인 작업을 수행할 수 있다.
* ApplicationArguments를 사용할 수 있다.
* 여러 Runner들의 실행 순서 결정을 위해 @Order를 정할 수 있다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
