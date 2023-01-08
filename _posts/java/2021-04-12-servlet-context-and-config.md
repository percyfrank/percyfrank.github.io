---
title: "ServletContext 및 ServletConfig"
excerpt: "Servlet을 초기화하는 핵심 개념들에 대해 알아보자."
categories:
  - Java
tags:
  - Java
date: 2021-04-12
last_modified_at: 2021-04-12
---

> [실습 Repository](https://github.com/xlffm3/servlet-practice)

## 1. ServletContext

> web.xml

```xml
<context-param>
    <description>Servlet Context Test</description>
    <param-name>lastname</param-name>
    <param-value>jinhong from Servlet Context</param-value>
</context-param>
```

웹 어플리케이션 전역에서 사용할 공동의 자원을 미리 바인딩하여 Servlet들이 이를 공유할 수 있도록 한다.

* Tomcat 컨테이너가 실행되면 웹 어플리케이션당 하나의 Context 객체를 생성한다.
* Servlet들끼리 자원(데이터)을 공유하는 게시판처럼 사용된다.
* Servlet과 컨테이너 간의 연동을 위해 사용한다.

<br>

## 2. ServletConfig

> web.xml

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.example.servlet.HelloServlet</servlet-class>
    <init-param>
        <param-name>firstname</param-name>
        <param-value>park from Servlet Config</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello-servlet</url-pattern>
</servlet-mapping>
```

ServletContext는 범위를 어플리케이션으로 한다면, ServletConfig는 Servlet 내부로 한정된다. Servlet 배포 시 설정된 정보를 Servlet으로 넘겨주기 위해서 사용된다. Servlet 안에 하드코딩하길 원하지 않는 정보들이며, 마찬가지로 DD에서 설정한다.

* 상위 Servlet 인터페이스의 메서드를 보면 ``init(ServletConfig servletConfig)``가 존재한다.
  * 즉, Servlet을 초기화하는데 ServletConfig가 사용된다.
* Servlet Container는 DD를 읽어 ServletConfig 인스턴스를 만든다.
  * Servlet당 ServletConfig 인스턴스를 하나씩 만든다.
* Servlet 클래스의 인스턴스를 생성하고 나서 ``init()`` 메서드를 호출하여 ServletConfig를 Servlet 인스턴스에게 제공한다.
  * Container가 ServletConfig를 해당 Servlet의 생명 주기동안 한 번만 읽기 때문에, Servlet이 살아있는 동안에는 수정할 수가 없다.

### 2.1. 초기화 메서드의 재정의

개발자가 재정의하는 Servlet의 초기화 메서드인 ``init()``는 매개변수가 없다.

> Servlet.java

```java
init(ServletConfig config){
    ...
    init();
    ...
}
```

* 실제로는 Servlet Container가 자동으로 ``init(ServletConfig servletConfig)``를 실행한다.
* 해당 메서드의 내부에서 개발자가 재정의한 ``init()``도 실행하게 된다.
* 즉, 개발자는 재정의할 내용이 있다면 ``init()``에만 재정의하면 된다.

<br>

## 3. 예제

> HelloServlet.java

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
     //생략
     PrintWriter out = response.getWriter();
     String lastname = this.getServletContext().getInitParameter("lastname");
     String firstname = this.getServletConfig().getInitParameter("firstname");

     out.println("<html><body>");
     out.println("<h1>" + message + helloService.getHelloName() + "</h1>");
     out.println("<h1>" + lastname + "</h1>");
     out.println("<h1>" + firstname + "</h1>");
     out.println("</body></html>");
 }
```

![image](https://user-images.githubusercontent.com/56240505/122165980-f502c780-ceb3-11eb-9c68-d48d5f02ae65.png)

<br>

---

## References

* [내장 객체 / ServletContext / ServletConfig](https://ecsimsw.tistory.com/entry/ServletContext-ServletConfig)
* [5. 속성과 리스너.md](https://github.com/binghe819/TIL/blob/master/Spring/Servlet/Head%20First%20Servlets%20%26%20JSP/5.%20%EC%86%8D%EC%84%B1%EA%B3%BC%20%EB%A6%AC%EC%8A%A4%EB%84%88.md#1-servletconfig)
