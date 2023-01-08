---
title: "[Spring Web MVC] 6장 : Spring MVC 설정"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:06:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. 기본적인 Spring MVC 설정

*	DispatcherServlet의 기본 전략에는 다소 한계가 있다.
*	@Configuration을 사용한 Java 설정 파일에 Spring 구성 요소를 직접 @Bean으로 사용해서 등록한다.

<br>

## 2. @EnableWebMvc

> WebConfig.java

```java
@Configuration
@ComponentScan
@EnableWebMvc
public class WebConfig {

    //Bean 등록
}
```

> WebApplication.java

```java
public class WebApplication implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.setServletContext(servletContext);
        context.register(WebConfig.class);
        context.refresh();
        DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic app = servletContext.addServlet("app", dispatcherServlet);
        app.addMapping("/app/*");
    }
}
```

*	Annotation 기반 Spring MVC를 사용할 때 편리한 웹 MVC 기본 설정이다.
*	DelegatingWebMvcConfiguration.class를 Import하는데, 상속 관계를 따라 올라가다 보면 실질적으로 사용할 Bean들이 등록되어 있다.
*	`context.setServletContext(servletContext);` 이 필요하다.

<br>

## 3. WebMvcConfigurer

> WebConfig.java

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/", ".jsp");
    }
}
```

*	@EnableWebMvc가 제공하는 Bean을 커스터마이징할 수 있는 기능을 제공하는 인터페이스이다.

<br>

## 4. Spring Boot MVC 설정

* Spring Boot의 “주관”이 적용된 자동 설정이 동작한다.
	* JSP 보다 Thymeleaf를 선호한다.
	* JSON을 지원한다.
	* 웰컴 페이지 및 파비콘 등 정적 리소스를 지원한다.

<br>

## 5. Spring MVC 커스터마이징

> application.properties

```properties
spring.mvc.view.prefix=/WEB-INF/jsp
spring.mvc.view.suffix=.jsp
```

* application.properties에서 여러 설정을 할 수 있다.
* 특히 Formatter 및 Converter 등을 WebMvcConfigurer 인터페이스를 구현하여 등록할 수 있으나, Bean만 등록하면 자동으로 이용이 가능하다.
* @Configuration + Implements WebMvcConfigurer : Spring Boot의 MVC 자동 설정 및 추가 설정을 위한 형태이다.
* @Configuration + @EnableWebMvc + Imlements WebMvcConfigurer : Spring Boot의 MVC 자동 설정을 사용하지 않는다.
  * @EnableWebMvc를 통해 import하는 클래스가 Spring Boot Auto Configuration 조건에 위배되기 때문이다.
* 가급적 application.properties를 먼저 고려해보고, WebMvcConfigurer를 사용해도 원하는 기능을 구현하기 어려울 때 직접 Bean을 등록한다.

<br>

## 6. Spring Boot에서 JSP 사용하기

> “If possible, JSPs should be avoided. There are several known limitations when using them with embedded servlet containers.”

* JAR 프로젝트로 만들 수 없어 WAR 프로젝트로 만들어야 한다.
* Java -JAR로 실행할 수는 있지만 “실행가능한 JAR 파일”은 지원하지 않는다.
* 언더토우(JBoss에서 만든 서블릿 컨테이너)는 JSP를 지원하지 않는다.
* Whitelabel 에러 페이지를 error.jsp로 오버라이딩 할 수 없다.
* `./mvnw package` 로 메이븐 빌드를 할 수 있다.
* WAR 파일은 웹 서버에서 배포가 가능하다.

<br>

## 7. WAR 배포하기

![JAR](https://user-images.githubusercontent.com/56240505/80369855-e9c49500-88c9-11ea-8c61-b90dd5e63a8f.png)
![WAR](https://user-images.githubusercontent.com/56240505/80369899-fcd76500-88c9-11ea-88a9-89c8713663b1.png)

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
