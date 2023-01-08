---
title: "Spring MVC 및 Dispatcher Servlet"
excerpt: "Dispatcher Servlet이란?"
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-15
last_modified_at: 2021-06-15
---

> [실습 Repository](https://github.com/xlffm3/servlet-practice)

## 1. 기존 Java EE 프로젝트의 단점

> HelloServlet.java

```java
public class HelloServlet extends HttpServlet {

    private String message;

    public void init() {
        message = "Hello World!";
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        // Hello
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>" + message + "</h1>");
        out.println("</body></html>");
    }

    public void destroy() {
    }
}
```

> web.xml

```xml
<!-- 서블릿1 등록 -->
<servlet>
    <servlet-name>서블릿1</servlet-name>
    ...
</servlet>
<servlet-mapping>
    <servlet-name>서블릿1</servlet-name>
    <url-pattern>/1</url-pattern>
    ...
</servlet-mapping>

<!-- 서블릿2 등록 -->
<servlet>
    <servlet-name>서블릿2</servlet-name>
    ...
</servlet>
<servlet-mapping>
    <servlet-name>서블릿2</servlet-name>
    <url-pattern>/2</url-pattern>
    ...
</servlet-mapping>

<!-- 서블릿3 등록 -->
<servlet>
    <servlet-name>서블릿3</servlet-name>
    ...
</servlet>
<servlet-mapping>
    <servlet-name>서블릿3</servlet-name>
    <url-pattern>/3</url-pattern>
    ...
</servlet-mapping>
```

* Servlet 클래스를 작성하고 나서, ``web.xml``에 해당 Servlet과 URL을 맵핑해야 한다.
  * URL 하나당 새로운 Servlet 설정을 추가해야하는 번거로움이 존재한다.
* Servlet 클래스는 HttpServlet를 확장해야 한다.
  * HttpServlet 기능을 필수로 Override해야 하고, 더이상 일반 객체로 사용할 수 없다.
  * 즉, 클래스끼리 값을 주고 받기가 까다롭다.
* 모든 Servlet이 공통으로 처리할 혹은 가장 우선시 되어야 하는 작업을 개별적인 Servlet 객체가 수행할 수 없는 구조다.

<br>

## 2. Dispatcher Servlet

![image](https://user-images.githubusercontent.com/56240505/122085875-20e36600-ce3e-11eb-81fa-fa32dedef962.png)

> Servlet Container에서 HTTP프로토콜을 통해 들어오는 모든 요청을 프레젠테이션 계층의 제일 앞에 둬서 중앙집중식으로 처리해주는 Front Controller다.

클라이언트로부터 요청이 오면 Tomcat과 같은 Servlet Container가 요청을 받는다. 이 때, 제일 앞에서 서버로 들어오는 모든 요청을 처리하는 프론트 컨트롤러를 Spring이 Dispatcher Servlet으로 정의했다. 공통 처리 작업을 Dispatcher Servlet이 처리한 후, 적절한 세부 @Controller(핸들러)로 작업을 위임한다.

### 2.1. Flow

1. Servlet 필터가 존재한다면 모든 요청에 대한 서블릿 필터가 실행된다.
2. 모든 요청은 DispatcherServlet에게 전달된다.
3. DispatcherServlet은 받은 요청을 분석하여, 공통적으로 처리할 부분을 처리한다.
  * Local, Multipart, Theme 등.
4. DispatcherServlet 은 HandlerMapping 에게 위임하여 요청을 처리할 Controller(Handler)를 찾는다.
  * HandlerMapping은 요청 URL을 보고 Handler를 찾아서 Handler 의 이름과 함께 반환한다.
  * 반환되는 것은 HandlerExecutionChain 타입이다.
    * Handler와 인터셉터 관련 상태를 가진다.
5. DispatcherServlet 은 찾아낸 Handler를 실행할 수 있는 HandlerAdapter를 찾는다.
6. 찾아낸 HandlerAdapter를 사용해서 Handler를 실행한다.
  * 실행 전에 전처리, 후처리로 실행해야 할 인터셉터 목록을 결정하고 실행시킨다.
  * Handler를 실행하면서 비즈니스 로직 또한 실행한다.
  * Handler의 리턴값을 보고 어떻게 처리할지 판단한다.
    * Handler의 리턴값 : View(뷰의 파일명), Model(비즈니스 로직 처리한 후의 데이터)
    * 리턴되는 ModelAndView가 null이고 @ResponseEntity가 존재하면 View Name을 Resolve하지 않고 MessageConverter를 통해 응답 본문을 만든다.
7. DispatcherServlet은 ViewResolver에게 View의 이름을 전달하고 실제 리소스 View 객체를 얻는다.
  * View Resolver는 전략 객체이며 View Name뿐 아니라 헤더 정보(accept)도 전달된다.
  * View Resolver는 전달된 정보를 바탕으로 사용자에게 보여줄 View가 무엇인지 결정한다.
8. DispatcherServlet은 View 객체에게 Model과 함께 화면 표시를 의뢰한다.
9. View 객체는 해당하는 View(JSP, Thymeleaf 등)를 호출하며, Model 객체에서 화면 표시에 필요한 정보를 가져와 화면 표시를 처리한다.
  * 찾은 View에 Model 데이터를 랜더링하고 요청의 응답값을 생성한다.
10. (부가적으로) 예외가 발생했다면, 예외 처리 핸들러에 요청 처리를 위임한다.
11. DispatcherServlet은 View로부터 받은 결과를 클라이언트에게 반환한다.

<br>

## 3. Servlet 어플리케이션에 Spring 연동하기

### 3.1. 용어 정리

객체는 ``init()`` 메서드 호출을 통해 Servlet이 된다.

1. ServletConfig
  * Servlet 배포 시 설정된 정보를 Servlet으로 넘겨주기 위해서 사용된다.
  * Servlet 당 ServletConfig 객체 하나가 존재한다.
  * Servlet 안에 하드코딩하길 원하지 않는 정보들이다.
  * DB 혹은 EJB 참조 이름 등.
  * ServletContext에 접근하기 위해 이 객체를 사용하며, 파라미터 값은 배포 서술자(DD)에서 설정 가능하다.
  * ``<init-param>``
2. ServletContext
  * 웹 애플리케이션 당 하나의 ServletContext가 있다.
  * 이름을 AppContext라고 해야 한다.
  * 웹 애플리케이션의 파라미터 정보를 읽어오기 위하여 사용된다.
  * 파라미터 정보는 DD안에 설정되어 있다.
  * 메시지(속성)를 적어놓으면 애플리케이션의 다른 부분들이 이를 읽을 수 있게된다.
  * Servlet들끼리 자원(데이터)을 공유하는 게시판처럼 사용된다.
  * Servlet과 컨테이너 간의 연동을 위해 사용한다.
  * 컨테이너의 이름과 버전 및 지원하는 API 버전 등 서버 정보를 파악하기 위하여 사용된다.

### 3.2. 의존성 설정

> pom.xml

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

> web.xml

```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

### 3.3. ContextLoaderListener

> ContextLoaderListener.java

```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
```

ContextLoaderListener는 ServletContextListener의 구현체다.

* ServletContextListener란 ServletContext의 초기화 이벤트를 리스닝하는 것이다.
  * 웹 어플리케이션이 처음 구동될 때, ServletContext에 저장되어 있는 정보를 바탕으로 필요한 작업을 셋팅함으로써 모든 Servlet이 공유할 수 있게 하는 효율적인 구조화가 가능하다.
  * 예를 들어, ServletContext에 저장되어 있는 DB 정보를 가져와 DB Connection 객체를 생성하는 등이 있겠다.
* Context 초기화 파라미터를 읽고, 어플리케이션이 클라이언트에게 서비스하기 전에 특정 코드를 실행할 수 있다.
  * 이 때 ServletContext라는 이벤트가 발생하고, Listener가 특정 코드를 실행시키는 것이다.
  * DB Connection과 웹 어플리케이션에 필요한 공용 객체 등.

> ContextLoader.java

```java
 public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
}
```

ServletContext 생성시 Spring의 WebApplicationContext를 생성한다.

* Spring의 WebApplicationContext를 ServletContext의 속성으로 등록한다.
  * ApplicationContext(Spring IoC Container)를 생성하여 ServletContext에 바인딩한다.
  * Tomcat이 생성하는 WebApplicationContext(Servlet Container)에 등록된 Servlet들이 Spring IoC Container를 사용할 수 있다.
* ApplicationContext를 ServletContext 생명주기에 따라 등록하고 소멸한다.
  * ``contextInitialized()`` 및 ``contextDestroyed()`` 메서드를 확인하자.
* Servlet에서 IoC 컨테이너를 ServletContext를 통해 꺼내 사용한다.
* **따라서 ContextLoaderListener는 XML이나 Java Configruation 등의 별도의 설정 메타 정보를 바탕으로 ApplicationContext를 생성해야 한다.**

> web.xml

```xml
<context-param>
    <param-name>contextClass</param-name>
    <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
</context-param>
<context-param>
    <param-name>contextConfigLocation</param-name> <!-- 스프링 설정 파일 변수 -->
    <param-value>com.example.servlet.AppConfig</param-value> <!-- 스프링 설정 파일의 위치 -->
</context-param>
```

> AppConfig.java

```java
@Configuration
@ComponentScan
public class AppConfig {
}
```

* Spring은 WAS 동작시 스캔되는 Context 속성 값을 기반으로 Bean을 관리하는 ApplicationContext를 만든다.
  * 위 코드에선 애노테이션의 자바 설정 기반의 IoC 컨테이너(AnnotationConfigWebApplicationContext)를 만들도록 설정한다.

> HelloService.java

```java
@Component
public class HelloService {

    public String getHelloName() {
        return "Wooteco";
    }
}
```

> HelloServlet.java

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
    response.setContentType("text/html");

    ApplicationContext applicationContext = (ApplicationContext) request.getServletContext().getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
    System.out.println(applicationContext.getClass());
    //class org.springframework.web.context.support.AnnotationConfigWebApplicationContext

    HelloService helloService = applicationContext.getBean("helloService", HelloService.class);
    // Hello
    PrintWriter out = response.getWriter();
    out.println("<html><body>");
    out.println("<h1>" + message + helloService.getHelloName() + "</h1>");
    out.println("</body></html>");
}
```

![image](https://user-images.githubusercontent.com/56240505/122145143-a6433680-ce8f-11eb-8c21-b09c0ae14ebe.png)

* 정리하자면 contextClass에 등록된 WebApplicationContext(XML 혹은 Annotation)를 통해 IoC 컨테이너를 생성한다.
* contextConfigLocation에 등록된 정보를 바탕으로 Bean을 만든다.
* 등록된 IoC 컨테이너 (ApplicationContext)가 바로 RootWebApplicationContext다.

<br>

## 4. DispatcherServlet 연동

![image](https://user-images.githubusercontent.com/56240505/122145804-e48d2580-ce90-11eb-805e-13a02e5bc462.png)

> DispatcherServlet은 IoC를 속성으로 등록하며, 웹에 필요한 Bean들을 모두 관리한다. 또한 모든 요청을 Dispatch한다.

xml 설정으로 등록된 Listener가 Root WebApplicationContext를 만들더라도, DispatcherServlet은 Root ApplicationContext를 상속받는 Servlet WebApplicationContext를 새로 만든다.

* Root WebApplicationContext는 다른 Servlet도 공용으로 사용하고, DispatcherServlet이 생성한 WebApplicationContext은 해당 DispatcherServlet만 사용한다.
  * Root WebApplicationContext는 다른 DispatcherServlet도 사용하기 때문에 Web과 관련된 Bean은 없고 Service 및 Repository 등 공용 자원들이 존재한다.
  * Servlet WebApplicationContext에는 해당 DispatcherServlet에 한정적인 Web 관련 Bean이 존재한다.
* WebApplicationContext는 연관된 ServletContext 및 Servlet에 대해 참조를 가지고 있다.
* DispatcherServlet마다 자체적으로 Controller, HandlerAdapter 등등의 Bean들을 따로 가지고 있는다.
  * 상속을 통해 동일하게 IoC 컨테이너를 사용한다.

즉, 정리하자면 다음과 같다.

* ServletContext에 등록된 IoC Container : Root WebApplicationContext
* ServletConfig에 등록된 IoC Container : Servlet WebApplicationContext

### 4.1. 설정 분리

> AppConfig.java

```java
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(Controller.class))
public class AppConfig {
}
```

> WebConfig.java

```java
@Configuration
@ComponentScan(useDefaultFilters = false, includeFilters = @ComponentScan.Filter(Controller.class))
public class WebConfig {
}
```

두 Context의 설정 파일을 분리한 이유는 스코프가 다르기 때문이다.

* Root WebApplicationContext는 Spring IoC 컨테이너를 의미하며 공유 자원을 관장한다.
  * Servlet Context에 저장하므로 여러 DispatcherServlet 에서 모두 접근이 가능하다.
  * Spring IoC Container는 웹과 관련이 없는 비즈니스 로직(Service, Repository)을 저장한다.
* Servlet WebApplicationContext는 DispatcherServlet의 고유의 웹 자원을 의미한다.
  * ServletConfig에 저장하므로 다른 DispatcherServlet은 접근이 불가하다.
  * DispatcherServlet에는 웹과 관련된 Controller, View Resolver 등을 저장한다.

> web.xml

```xml
<servlet>
    <servlet-name>Dispatcher Servlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 따로 설정해주는 부분 -->
    <init-param>
        <param-name>contextClass</param-name>
        <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.example.servlet.WebConfig</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>Dispatcher Servlet</servlet-name>
    <url-pattern>/app/*</url-pattern>
</servlet-mapping>
```

* 별도로 DispatcherServlet을 등록한다.
  * contextClass : 어떠한 컨텍스트 타입을 사용할 것인가?
  * contextConfigLocation : 어떤 설정 파일을 읽을 것인가?
* ``/app`` 이하로 들어오는 모든 요청을 DispatcherServlet에게 요청하도록 한다.

### 4.2. 테스트

> HelloController.java

```java
@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;

    @GetMapping("/dispatcher")
    public String testDispatcherServlet() {
        return "jinhong " + helloService.getHelloName();
    }
}
```

![image](https://user-images.githubusercontent.com/56240505/122146929-d8a26300-ce92-11eb-9d4d-11f522b6d420.png)

### 4.3. 중요한 점

Servlet WebApplicationContext는 Root WebApplicationContext를 상속받아 사용하므로, Controller에서 Service나 Repository의 의존성을 주입받을 수 있다.

* Root WebApplicationContext는 모든 Servlet에서 접근 및 공유해서 사용이 가능하다.
  * ServletContext에 저장하기 때문이다.
* Servlet WebApplicationContext는 해당 Servlet에서만 접근이 가능하다.
  * 즉, DispatcherServlet는 다른 DispatcherServlet의 애플리케이션 컨텍스트를 모른다.
  * ServletConfig에 저장하기 때문이다.

여러 DispatcherServlet을 만들고 공용으로 사용해야하는 Bean들을 공유하기 위해서 이런 상속 구조가 형성되었다고 한다. 굳이 상속관계를 만들지 않고 Servlet WebApplicationContext만을 등록해서 모든 Bean을 담아도 상관 없다.

* 오늘 날은 대부분이 Servlet WebApplicationContext(DispatcherServlet Config에 저장된 IoC 컨테이너)만으로 구성한다고 한다.
* 최근에는 DispatcherServlet이 1개인 경우가 대다수라 계층 구조를 크게 신경쓰는 일이 적다.

<br>

## 5. Spring MVC와 Spring Boot

### 5.1. Spring Boot를 사용하지 않는 Spring MVC

![image](https://user-images.githubusercontent.com/56240505/80369899-fcd76500-88c9-11ea-88a9-89c8713663b1.png)

> WebXML.java

```java
public class WebXML implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(WebConfig.class);
        context.refresh();
        DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic app = servletContext.addServlet("app", dispatcherServlet);
        app.addMapping("/app/*");
    }
}
```

* Tomcat 등 Servlet Contianer에 등록한 Web ApplicationContext에 DispatcherServlet을 등록한다.
  * web.xml에 Servlet을 등록하기.
  * 혹은 WebApplicationInitializer에 Java 코드로 등록하기.
    * 세부 구성 요소는 @Configuration의 Bean을 설정하기 나름이다.
* Servlet Context 내부에 Spring이 들어가는 구조다.

### 5.2. Spring Boot를 활용한 Spring MVC

![image](https://user-images.githubusercontent.com/56240505/80369855-e9c49500-88c9-11ea-8c61-b90dd5e63a8f.png)

Java 어플리케이션에 내장 Tomcat을 만들고, 그 안에 DispatcherServlet을 등록한다. 이는 Spring Boot 자동 설정이 자동으로 해준다. Spring Boot의 주관에 따라 여러 인터페이스 구현체를 Bean으로 등록한다.

* DispatcherServlet에서 사용하는 여러 구성 요소들의 기본 값들이 일반 스프링 MVC보다 더 많이 정의되어 있다.
* DispatcherServlet에서 기본 null이었던 MultipartResolver가 자동으로 등록되어, Spring Boot에서는 별다른 설정없이 파일 업로드가 가능하는 등의 이점이 존재한다.

<br>

## 6. 정리

![image](https://user-images.githubusercontent.com/56240505/122148878-25d40400-ce96-11eb-83a1-b3f2adf04c90.png)

DispatcherServlet이란 Servlet Config에 등록되는 IoC Container + Dispatch 하는데 필요한 객체들 + Servlet 등 굉장히 복잡한 Servlet이라 볼 수 있다.

DispatcherServlet은 초기화할 때 자체적으로 IoC 컨테이너를 ServletConfig에 등록한다.

* HandlerMapping이나 HandlerAdapter 등 필요한 Bean을 가져와 인스턴스 변수로 등록해둔다.
* ``DispatcherServlet.properties``에 빈으로 등록되는 설정 내용을 볼 수 있다.

<br>

---

## References

* 스프링 웹 MVC(백기선, Inflearn)
* [서블릿에 스프링 연동.md](https://github.com/binghe819/TIL/blob/master/Spring/MVC/%EC%84%9C%EB%B8%94%EB%A6%BF%EC%97%90%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%97%B0%EB%8F%99.md)
