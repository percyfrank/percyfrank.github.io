---
title: "[Spring Web MVC] 2장 : Servlet"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:02:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. Servlet

* Java 엔터프라이즈 에디션(EE)은 웹 애플리케이션 개발용 스펙 API를 제공한다.
* 요청 당 쓰레드를 사용한다.
* 그 중에 가장 중요한 클래스중 하나가 HttpServlet이다.
* Servlet 등장 이전에 사용하던 기술인 CGI(Common Gateway Interface)는 요청 당 프로세스를 만들어 사용했다.
* 따라서 Servlet은 CGI에 비해 빠르고, 플랫폼 독립적이며 보안 및 이식성이 우수하다.

<br>

## 2. Servlet Engine or Container

* 세션 관리.
* 네트워크 서비스.
* MIME 기반 메시지 인코딩 디코딩.
* Servlet 생명주기 관리.
* ...

<br>

## 3. Servlet Lifecycle

* Servlet 컨테이너가 Servlet 인스턴스(HttpServlet 상속)의 `init()` 메소드를 호출하여 초기화 한다.
* 최초 요청을 받았을 때 한번 초기화 하고 나면 그 다음 요청부터는 이 과정을 생략한다.
* Servlet이 초기화 된 다음부터 클라이언트의 요청을 처리할 수 있다.
* 각 요청은 별도의 쓰레드로 처리하고 이때 Servlet 인스턴스의 `service()` 메소드를 호출한다.
  * 이 안에서 HTTP 요청을 받고 클라이언트로 보낼 HTTP 응답을 만든다.
  * `service()`는 보통 HTTP Method에 따라 `doGet()`, `doPost()` 등으로 처리를 위임한다.
  * 따라서 보통 `doGet()` 또는 `doPost()`를 구현한다.
* Servlet 컨테이너 판단에 따라 해당 Servlet을 메모리에서 내려야 할 시점에 `destroy()`를 호출한다.
* [정리 자료](https://smujihoon.tistory.com/167)

<br>

## 4. Servlet Listener

* 웹 애플리케이션에서 발생하는 주요 이벤트를 감지하고 각 이벤트에 특별한 작업이 필요한 경우에 사용할 수 있다.
* ServletContextListener 클래스를 상속받아 사용한다.
* 서블릿 컨텍스트 수준의 이벤트.
	* 컨텍스트 라이프사이클 이벤트.
	* 컨텍스트 애트리뷰트 변경 이벤트.
* 세션 수준의 이벤트.
	* 세션 라이프사이클 이벤트.
	* 세션 애트리뷰트 변경 이벤트.

<br>

## 5. Servlet Filter

![Servlet](https://user-images.githubusercontent.com/56240505/80300470-15675280-87d8-11ea-94d5-511dcae0448b.png)

* 들어온 요청을 서블릿으로 보내고, 또 서블릿이 작성한 응답을 클라이언트로 보내기 전에 특별한 처리가 필요한 경우에 사용할 수 있다.
* 체인 형태의 구조이다.
* Filter 클래스를 상속받아 사용한다.

<br>

## 6. Servlet 관련 코드

> web.xml

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <filter>
    <filter-name>myFilter</filter-name>
    <filter-class>com.glenn.MyFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>myFilter</filter-name>
    <servlet-name>hello</servlet-name>
  </filter-mapping>
  <listener>
    <listener-class>com.glenn.MyListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.glenn.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

> MyListener.java

```java
public class MyListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("context init");
        servletContextEvent.getServletContext().setAttribute("name", "jipark");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("context destoryed");
    }
}
```

> MyFilter.java

```java
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("filter init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("filter destroyed");
    }
}
```

* 컨텍스트-필터-서블릿 순으로 초기화되며, 역순으로 파괴된다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
