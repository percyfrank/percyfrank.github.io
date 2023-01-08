---
title: "[Spring Web MVC] 3장 : IoC 컨테이너 & MVC 연동"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:03:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. Spring IoC 컨테이너 연동

> pom.xml

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>5.1.3.RELEASE</version>
</dependency>
```

> web.xml

```xml
<context-param>
  <param-name>contextClass</param-name>
  <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>com.glenn.AppConfig</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

> AppConfig.java

```java
@Configuration
@ComponentScan
public class AppConfig {
}
```

> HelloService.java

```java
@Service
public class HelloService {

    public String getName() {
        return "glenn";
    }
}
```

> HelloServlet.java

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    ApplicationContext context = (ApplicationContext) getServletContext().getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
    HelloService helloService = (HelloService)context.getBean(HelloService.class);
    System.out.println("do get");
    resp.getWriter().println("<html><body><h1>hi" + helloService.getName() + "</h1></body></html>");
}
```

* ContextLoaderListener.
	* 서블릿 리스너 구현체이다.
	* ApplicationContext를 만들어 준다.
	* ApplicationContext를 서블릿 컨텍스트 라이프사이클에 따라 등록하고 소멸시켜준다.
	* 서블릿에서 IoC 컨테이너를 ServletContext를 통해 꺼내 사용할 수 있다.
* 이를 통해 Servlet들이 ApplicationContext(Spring IoC 컨테이너)와 등록된 Bean을 사용할 수 있다.
* Listener가 사용하는 여러 파라미터들을 XML에 지정해주어야 한다.
  * Servlet이 추가될 때 마다 xml 설정이 늘어나는데, 디자인 패턴 중 Front Controller로 이를 해결한다.

<br>

## 2. DispatcherServlet

![DispatcherServlet](https://user-images.githubusercontent.com/56240505/80301210-bb698b80-87dd-11ea-9809-f4d9f2dd3fb9.png)

* Spring MVC의 핵심이며, Front Controller의 역할을 한다.
* xml 설정으로 등록된 Listener가 Root WebApplicationContext를 만들더라도, DispatcherServlet은 Root ApplicationContext를 상속받는 WebApplicationContext를 새로 만든다.
* Root WebApplicationContext는 다른 Servlet도 공용으로 사용하고, DispatcherServlet이 생성한 WebApplicationContext은 해당 DispatcherServlet만 사용한다.
  * Scope이 다르기 때문에 이러한 상속 구조가 생겨났다.
* Root WebApplicationContext는 다른 DispatcherServlet도 사용하기 때문에 Web과 관련된 Bean은 없고 공용 자원들이 존재한다.
* Servlet WebApplicationContext에는 해당 DispatcherServlet에 한정적인 Web 관련 자원이 존재한다.
* WebApplicationContext는 연관된 ServletContext 및 Servlet에 대해 참조를 가지고 있다.

<br>

## 3. Spring MVC 연동

> web.xml

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
  </context-param>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.glenn.AppConfig</param-value>
  </context-param>  
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>app</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>com.glenn.WebConfig</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>app</servlet-name>
    <url-pattern>/app/*</url-pattern>
  </servlet-mapping>
</web-app>
```

> AppConfig.java

```java
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(Controller.class))
```

* Root WebApplicationContext에 등록될 설정들(Web과 관련된 Controller 제외)만을 Scan한다.

> WebConfig.java

```java
@Configuration
@ComponentScan(useDefaultFilters = false, includeFilters = @ComponentScan.Filter(Controller.class))
```

* Servlet WebApplicationContext에 등록될 설정들(Web과 관련된 Controller)만을 Scan한다.

> HelloController.java

```java
@RestController
public class HelloController {

    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public String hello() {
        return "mvc test hello!! " + helloService.getName();
    }
}
```

* 계층 구조를 만들 때, HelloService Bean은 ContextLoaderListener(Root WebApplicationContext)에 등록하고 HelloController는 DispatcherServlet의 WebApplicationContext에 등록한다.
  * Web 관련 설정은 DispatcherServlet가 관장하고, 다른 Servlet들도 공통으로 사용할 수 있는 자원은 ContextLoaderListener(Root)가 관장하도록 한다.
* ContextLoaderListener가 contextClass와 contextConfigLocation을 설정했듯, DispatcherServlet 또한 이를 설정해준다.
* 이후 @ComponentScan을 하는 Config에 각각 필터를 설정하여, ContextLoaderListener는 Controller를 제외하게 하고 DispatcherServlet은 Controller만 등록할 수 있게 한다.
* DispatcherServlet은 ContextLoaderListener의 WebApplicationContext를 부모로 가진다.
* @ComponentScan의 필터를 삭제한 뒤, ContextLoaderListener 관련 설정을 삭제하고 모든 Bean이 DispatcherServlet에 등록시켜 사용할 수 있다.
  * 최근에는 DispatcherServlet이 1개인 경우가 대다수라 계층 구조를 크게 신경쓰는 일이 적다.
* 이와 같은 코드는 서블릿 컨텍스트 내부에 스프링이 들어가는 구조이며, Spring Boot는 Spring이라는 Java 어플리케이션 내부에 톰캣이 들어가있는 구조이다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
