---
title: "Java EE, Servlet, JSP의 개념 및 역사"
excerpt: "Java EE 관련 개념 및 역사에 대해 알아보자."
categories:
  - Java
tags:
  - Java
date: 2021-04-10
last_modified_at: 2021-04-10
---

## 1. Java EE의 역사

### 1.1. Applet

초창기 Java는 Applet같은 클라이언트 GUI를 만드는데 초점이 맞춰져 있었습니다. Applet이란 Java 기반의 리치 인터넷 어플리케이션입니다. 리치 인터넷 어플리케이션은 어도비 플래시와 같이 웹 사이트에서 사용자 경험을 향상시키기 위한 프로그램이며, 주로 웹 브라우저의 부족한 멀티 미디어나 그래픽 성능 등을 보강하는 역할입니다.

### 1.2. J2EE

당시 기업용 서버 소프트웨어 개발은 C/C++ 및 다양한 회사의 미들웨어 제품을 사용해서 개발하는 방식이었습니다. 미들웨어란 컴퓨터 제작 회사가 사용자의 특정한 요구대로 만들어 제공하는 프로그램으로, OS와 응용 소프트웨어의 중간에서 조정과 중개의 역할을 수행하는 소프트웨어입니다.

이러한 방식으로 개발된 서버 소프트웨어는 사용 중인 OS 및 미들웨어 제품에 종속된다는 단점이 존재했습니다. 이 때, 개발자들은 Java의 서버 시장에서의 가능성에 주목하기 시작했습니다. JVM을 활용한 Java의 플랫폼 독립적인 특성을 활용해 미들웨어에 필요한 공통 API를 제공하면, 특정 미들웨어 제품에 종속되는 단점을 극복할 수 있기 때문입니다.

서버 개발에 필요한 기능을 모아 J2EE(Java 2 Enterprise Edition)이라는 표준을 만들고, 각 기업들은 해당 표준을 준수하는 미들웨어 제품을 판매하기 시작했습니다. 개발자들은 이제 어느 제품을 사용하더라도 API를 새로 공부할 필요가 없어졌습니다. 또한 서버 소프트웨어를 미들웨어 제품 및 OS에 따라 포팅할 필요가 없어졌습니다.

### 1.3. Java EE

정리하자면 J2EE는 Java를 이용하여 서버 측을 개발하는 플랫폼입니다. J2EE는 PC에서 동작하는 표준 플랫폼인 Java SE의 확장 버전으로서, 웹 어플리케이션 서버에서 동작하는 장애 복구 및 분산 멀티 티어 등 추가 기능을 탑재했습니다. 5.0 버전 이후로는 Java EE로 개칭되었습니다.

Java EE는 Servlet과 JSP를 만들고 실행하는 규칙 및 EJB(Enterprise Java Beans)의 분산 컴포넌트와 웹 서비스 규칙 등을 추가로 가지고 있습니다. Web(Servlet) Container와 EJB Container를 가지고 있다고 생각하면 이해하기 쉽습니다.

이러한 Java EE 스펙에 따라 제품으로 구현한 것을 웹 어플리케이션 서버 또는 WAS라고 부릅니다. 기업들이 WebSphere 등 Java EE에 호환되는 어플리케이션 서버 제품을 앞다투어 출시하게 시작했습니다. 특히 웹 개발을 위해 Java EE 표준에 포함된 Servlet과 JSP는 당시 유행하던 PHP와 CGI를 제치고 큰 인기를 얻었습니다.

<br>

## 2. 동적 컨텐츠와 CGI

> 구체적인 Web Server와 WAS의 차이점 및 특징은 [여기](https://xlffm3.github.io/network/was-webserver/)를 참조해주세요.

![image](https://user-images.githubusercontent.com/56240505/122050353-5b3d0b00-ce1e-11eb-9f0c-5441d6d376ba.png)

과거의 웹 어플리케이션은 HTML, CSS, Image 등 정적으로 준비된 컨텐츠를 Web Server를 통해 클라이언트에게 제공했습니다. 그러나 웹의 보급으로 인해 클라이언트 요청에 맞게 개인화된 콘텐츠를 동적으로 생성해서 보여줄 필요성이 증대되었습니다.

![image](https://user-images.githubusercontent.com/56240505/122050412-65f7a000-ce1e-11eb-9dd0-2572b5d8b368.png)

Web Server는 연산이 필요없는 정적인 컨텐츠를 주로 처리하고, DB 조회나 다양한 로직 처리를 요구하는 연산이 필요한 경우 WAS에게 요청 객체를 넘깁니다. WAS는 요청에 대해 로직대로 연산을 수행한 뒤 응답 결과(동적인 컨텐츠)를 생성해 Web Server(->클라이언트)로 반환합니다.

정리하자면 Web Server는 HTTP 프로토콜 레벨의 서비스를 제공하고, WAS는 더 강력한 동적 웹 서비스를 제공합니다.

### 2.1. CGI(Common Gateway Interface)

![image](https://user-images.githubusercontent.com/56240505/121856488-81d04880-cd2f-11eb-96a4-d2471eac9750.png)

CGI(Common Gateway Interface)는 Web Server와 어플리케이션 간에 데이터를 주고 받는 방식 또는 컨벤션입니다. 클라이언트로부터 요청을 받는 Web Server와 요청을 받아 처리해줄 로직을 담은 어플리케이션 프로그램 사이의 인터페이스입니다. Web Server가 특정 언어로 쓰인 구체적인 프로그램이 아니라 인터페이스에 의존하기에, 해당 인터페이스를 구현하면(CGI 스펙을 따른다면) 구현체에 상관없이 어플리케이션과 Web Server가 소통할 수 있습니다.

CGI를 사용하는 프로그램은 클라이언트 요청 내용에 맞게 HTML을 생성하는 등 동적 컨텐츠 생성이 가능합니다. 그러나 CGI는 요청마다 새로운 프로세스를 만들기에, 대량의 액세스가 있을 때 Web Server에 부하가 심해집니다.

<br>

## 3. Servlet

Servlet이란 Java를 사용하여 웹 페이지를 동적으로 생성하는 서버측 프로그램 혹은 그 기술을 일컫습니다. 클라이언트의 요청을 처리하고 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 Java 웹 프로그래밍 기술입니다. 쉽게 말해 Java로 구현된 CGI입니다.

### 3.1. 특징

1. 클라이언트의 요청에 대해 동적으로 작동하는 웹 어플리케이션 컴포넌트입니다.
2. 요청마다 새로운 Java Thread를 만들어 동작합니다.
  * CGI보다 부하가 적습니다.
3. HTTP 프로토콜 서비스를 지원하는 ``javax.servlet.http.HttpServlet`` 클래스를 상속받습니다.
4. HTML 변경 시 Servlet을 재컴파일해야하는 단점이 존재합니다.

### 3.2. 동작 방식

![image](https://user-images.githubusercontent.com/56240505/122054422-9e997880-ce22-11eb-80c5-58b0b64ccbc4.png)
![image](https://user-images.githubusercontent.com/56240505/122054437-a2c59600-ce22-11eb-8263-9ffb2e05f4e7.png)
![image](https://user-images.githubusercontent.com/56240505/122054448-a78a4a00-ce22-11eb-999a-ec59ad1af280.png)
![image](https://user-images.githubusercontent.com/56240505/122056147-495e6680-ce24-11eb-825f-8afd21f7345b.png)

1. 클라이언트가 URL을 입력하면 HTTP Request가 Servlet Container로 전송합니다.
2. 요청을 전송받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성합니다.
3. web.xml을 기반으로 사용자가 요청한 URL이 어느 서블릿에 대한 요청인지 찾습니다.
4. 해당 서블릿에서 service메소드를 호출한 후 클리아언트의 GET, POST여부에 따라 ``doGet()`` 또는 ``doPost()``를 호출합니다.
5. 메서드 호출을 통해 동적 페이지를 생성한 후 HttpServletResponse객체에 응답을 보냅니다.
6. 응답이 끝나면 HttpServletRequest, HttpServletResponse 두 객체를 소멸시킵니다.

### 3.3. 예제

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
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.demo.javaee.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
  </servlet-mapping>
</web-app>
```

* 위와 같은 설정을 담은 Java EE 프로젝트를 WAR 패키징한 뒤, 미리 설치된 Tomcat 등의 Servlet Container에 WAR 파일을 올려 배포할 수 있습니다.
* ``http://localhost:8080/hello``로 GET 요청을 보내면 어플리케이션이 동적으로 만든 화면이 브라우저에 렌더됩니다.

<br>

## 4. Servlet Container

Servlet의 실행 환경을 Servlet Container라고 합니다. Servlet Container는 Servlet을 관리하는 역할을 합니다. 클라이언트의 요청을 받아 응답할 수 있도록, Web Server와 소켓으로 통신합니다. Servlet과 JSP를 위한 런타임 환경을 제공합니다.

### 4.1. Tomcat

> WAS는 EJB, JMS, CDI, JTA, Servlet API 등 Java EE의 전체를 지원합니다.

Servlet Container는 Web Container 혹은 Servlet Engine이라고도 불립니다. Tomcat은 WAS에 필수적인 EJB 같은 기술이 적용되어 있지 않아, 엄밀하게 말하자면 WAS가 아닌 Servlet Container에 가깝습니다.

후술할 Spring이 J2EE의 EJB 영역을 대체하면서, Tomcat 또한 포괄적으로 WAS라고 칭해도 문제는 없습니다. 또한 1개의 WAS는 여러 개의 컨테이너를 구성해서 각각 독립적인 서비스로 구동시키는 것도 가능합니다. 다만 하나의 WAS에 하나의 컨테이너만 사용한다면 WAS와 컨테이너를 구분지어 생각할 필요는 없습니다.

또한 Tomcat에도 기본적인 웹 서버 기능이 있으며, Apache Tomcat이란 경량 Apache Web Server와 Tomcat의 조합이라 생각하면 쉽습니다. 다만 실제 Apache 웹 서버에 비해 기능이 한정적이라 규모가 있는 웹 서비스는 이 둘을 분리해 사용합니다.

### 4.2. 특징

1. Web Server와의 통신 지원
  * Servlet Container는 Web Server와 Servlet간의 웹 소켓 통신을 API로 제공함으로써, 복잡한 통신 구축 과정을 생략합니다.
  * 따라서 개발자는 Servlet에 구현해야 할 비즈니스 로직에만 집중할 수 있습니다.
2. Servlet 생명 주기 관리
  * Servlet Container는 Servlet 클래스를 로딩하여 인스턴스화하고 초기화 메서드를 호출합니다.
  * 요청이 들어오면 적절한 메서드를 호출합니다.
  * Servlet을 적절한 순간에 GC를 진행하여 제거합니다.
3. 멀티 쓰레드 지원 및 관리
  * Servlet Container는 요청이 올 때 마다 새로운 Thread를 생성합니다.
  * HTTP Service 메서드를 실행하면 해당 Thread는 소멸합니다.
  * Servlet Container가 멀티 쓰레드를 지원 및 운영하기에, 개발자가 특별히 쓰레드 안전성을 관리할 필요가 없습니다.
4. 선언적인 보안 관리
  * 보안과 관련된 내용을 Servlet 또는 Java 클래스에 구현하지 않고, 일반적으로 XML 배포 서술자에 기록합니다.
  * 보안에 대한 수정이 생겨도 Java 소스 코드 수정 및 재컴파일하지 않아도 됩니다.

### 4.3. Servlet Life Cycle

![image](https://user-images.githubusercontent.com/56240505/122056996-1d8fb080-ce25-11eb-9dd8-c55e882a6748.png)

1. 클라이언트 요청이 들어오면 Servlet Container는 해당 Servlet이 메모리에 존재하는지 확인합니다.
2. 없을 경우, ``init()`` 메서드를 호출하여 적재합니다.
  * 객체가 Servlet이 되는 순간으로서, Servlet에 따라오는 고유한 권한을 가집니다.
    * 예를 들어, ServletContext를 가지고 컨테이너로부터 정보를 읽어올 수 있는 능력 등이 있습니다.
3. 이후 클라이언트의 요청에 따라서 ``service()`` 메서드를 통해 ``doGet()`` 혹은 ``doPost()``등을 호출합니다.
4. Servlet Container가 Servlet에 종료 요청을 하면 ``destroy()`` 메서드가 호출됩니다.

<br>

## 5. JSP

예제의 HelloServlet 클래스는 Java 소스 코드에 HTML 코드가 들어있는 형태입니다. 복잡한 레이아웃의 페이지를 구성하려면 Java 코드가 비대해지며, 어플리케이션 로직과 프레젠테이션 로직이 산재하여 가독성이 나빠집니다.

JSP(Java Server Page)는 Servlet의 확장된 기술로 HTML코드에 JAVA 코드를 삽입하여 사용하는 기술입니다. JSP는 Servlet을 사용하여 웹을 만들 때, 화면 인터페이스 구현이 워낙 까다로운 단점을 보완하기 위해 만들어진 스크립트 언어입니다. 이로써 어플리케이션 로직과 프레젠테이션 로직을 분리한 **MVC 패턴**과 유사해집니다.

> index.jsp

```html
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Insert title here</title>
</head>
<body>

<%
	for(int i = 0; i < 10; i++) {
		out.println(i);
	}
%>

</body>
</html>
```

### 5.1. 동작 방식

![image](https://user-images.githubusercontent.com/56240505/122058358-8166a900-ce26-11eb-96fa-b5c2366776a3.png)

1. JSP는 먼저 Servlet ``.java`` 파일로 변환됩니다.
2. 이렇게 변환된 Servlet 파일을 컴파일하여 ``.class``로 만든 뒤 실행합니다.
3. 실행 결과는 Java 언어가 사라진 순수한 HTML 코드가 되며, 최종적으로 클라이언트에게 반환됩니다.

<br>

## 6. Spring

### 6.1. EJB

Java EE의 핵심은 EJB(Enterprise Java Beans)입니다. EJB는 기업의 핵심 서비스를 만들기 위한 분산처리와 트랜잭션 및 보안 등을 지원하는 컴포넌트 모델입니다.

IT 시스템의 의존도가 높아지면서 어플리케이션의 비즈니스 로직이 복잡해짐과 동시에, ``트랜잭션, 상태관리, 멀티 쓰레딩, 리소스 풀링, 보안`` 등 여러 로우 레벨의 기술적인 처리가 요구되었습니다. 많은 사용자의 처리 요구를 빠르고 안정적이면서 확장 가능한 형태로 유지하기 위해서입니다. 문제는 어플리케이션 개발자가 이 모든 영역을 전문적으로 커버하기란 어렵습니다.

EJB의 비전은 **"애플리케이션 개발을 쉽게 만들어준다. 애플리케이션 개발자는 로우 레벨의 기술들에 관심을 가질 필요도 없다."**였습니다. EJB는 분산처리와 트랜잭션 및 보안 등 엔터프라이즈급 어플리케이션이 필요로 하는 엔터프라이즈 서비스들을 **어플리케이션 코드와 분리해 독립적인 서비스**로 사용할 수 있게 만드는 장점이 있습니다.

그러나 EJB는 널리 쓰였지만 다음과 같은 단점들이 존재합니다.

* 복잡한 프로그래밍 모델입니다.
  * 멀티 DB 분산 트랜잭션을 사용하지 않는 어플리케이션도 무거운 JTA 기반의 트랜잭션 관리 기능을 사용해야 했습니다.
  * EJB의 혜택을 얻기 위해 모든 기능이 다 필요하지 않더라도 고가의 WAS를 구입해야 했습니다.
* EJB 컴포넌트는 컨테이너 안에서만 동작할 수 있는 객체 구조입니다.
  * 자동화된 테스트가 매우 어렵거나 불가능합니다.
* EJB 스펙을 따르는 Bean들은 객체지향적이지 않아 상속 및 다형성 등의 혜택을 누리지 못합니다.
* 핵심 비즈니스 로직 및 도메인 객체에 특정 환경이나 기술 관련 코드가 과하게 침투합니다.
  * 특정 서버 환경이나 기술에 종속적인 코드가 됩니다.
* Java EE 서버에 배포하기 위해 상당 분량의 XML 설정이 수반됩니다.

Java EE는 특정 미들웨어 제품에 종속적인 기존 환경을 극복하기 위해 대두되었습니다. 그러나 EJB를 사용하면서 서버 소프트웨어가 다시금 특정 Java EE 서버 제품에 종속되는 아이러니가 발생하게 되었습니다.

### 6.2. Spring의 대두

Spring Framework는 이러한 문제점을 개선하기 위해 개발되었습니다.

1. 고가의 풀스택 Java EE 서버가 아닌 Tomcat과 같은 일반 Servlet Container에서도 엔터프라이즈급 웹 어플리케이션이 잘 구동됩니다.
  * 더이상 비싼 Java EE 서버를 구매하지 않아도 EJB보다 간편하게 EJB가 제공하던 선언적 트랜잭션과 보안 처리 및 분산 환경 지원 등 주요 기능을 사용할 수 있습니다.
2. 개발자들의 번거로움이 줄어들었습니다.
  * 각 Java EE 서버에 특화된 설정을 따로 공부할 필요가 없어졌습니다.
  * 사용하던 어플리케이션의 서버 제품을 바꿀 때 마다 포팅할 필요가 없어졌습니다.
  * Spring만 이용하면 제품에 관계없이 간단하게 배포가 가능하기 때문입니다.
3. POJO 기반의 프레임워크입니다.
  * [POJO](https://xlffm3.github.io/java/POJO_Bean/)는 Java EE 등 프레임워크의 코드가 침투하여 프레임워크에 종속되는 무거운 객체들을 만드는 것에 반발하며 나타난 용어입니다.
  * J2EE에 비해 특정 인터페이스를 구현하거나 상속받을 필요가 없어 기존 라이브러리를 지원하기에 용이하고 객체가 가볍습니다.
  * POJO를 이용한 애플리케이션 개발의 장점을 살리면서, EJB에서 제공하는 엔터프라이즈 서비스와 기술을 그대로 사용하도록 도와줍니다.
  * Spring을 사용하면 POJO로 어플리케이션을 만들고, 엔터프라이즈 서비스를 비침투적으로 POJO에 적용할 수 있습니다.

> 심지어 Spring Boot에는 Tomcat 등 별도의 WAS가 내장되어 있어서, 어플리케이션을 WAR 패키징하고 별도의 WAS를 설치한 뒤 WAS에 WAR 파일을 올리는 별도의 작업이 필요 없어졌습니다. 웹 어플리케이션을 jar 파일로 빌드하여 빠르고 독립적으로 실행할 수 있습니다.

<br>

---

## References

* [다시 하자 기초! 서블릿 라이프 사이클](https://smujihoon.tistory.com/167)
* [Tutorial: Your first Java EE application](https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-ee-application.html#source_code)
* [Apache HTTP Server? Apache Tomcat? 서버 바로 알기](https://woowacourse.github.io/javable/post/2021-05-24-apache-tomcat/)
* [자바EE의 역사 및 스프링과의 관계](https://okky.kr/article/415474)
* [[Java의 역사 #2] 웹과 함께 발전한 Java](https://steemit.com/kr/@stunstunstun/java-2-java)
* [Tomcat(톰캣), JSP, Servlet(서블릿)의 기본 개념 및 구조](https://codevang.tistory.com/191)
* [[JSP] 서블릿(Servlet)이란?](https://mangkyu.tistory.com/14)
* [Common Gateway Interface(CGI)란 무엇인가](https://live-everyday.tistory.com/197)
* [EJB&SPRING](https://blog.naver.com/ikhyun84/20086106665)
* [스프링 프레임워크 1 - POJO에 대하여](https://limmmee.tistory.com/8)
