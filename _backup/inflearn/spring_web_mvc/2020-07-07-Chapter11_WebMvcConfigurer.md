---
title: "[Spring Web MVC] 11장 : 기타 WebMvcConfigurer 설정 및 요약"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot  
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-15T08:02:00-05:00
---

## 1. CORS 설정

* Cross Origin 요청 처리 설정이다.
* 같은 도메인에서 온 요청이 아니더라도 처리를 허용하고 싶다면 설정한다.

<br>

## 2. 리턴 값 핸들러 설정

* 스프링 MVC가 제공하는 기본 리턴 값 핸들러 이외에 리턴 핸들러를 추가하고 싶을 때 설정한다.

<br>

## 3. 아큐먼트 리졸버 설정

* 스프링 MVC가 제공하는 기본 아규먼트 리졸버 이외에 커스텀한 아규먼트 리졸버를 추가하고 싶을 때 설정한다.

<br>

## 4. 뷰 컨트롤러

*	단순하게 요청 URL을 특정 뷰로 연결하고 싶을 때 사용할 수 있다.

<br>

## 5. 비동기 설정

* 비동기 요청 처리에 사용할 타임아웃이나 TaskExecutor를 설정할 수 있다.

<br>

## 6. 뷰 리졸버 설정

* 핸들러에서 리턴하는 뷰 이름에 해당하는 문자열을 View 인스턴스로 바꿔줄 뷰 리졸버를 설정한다.

<br>

## 7. Content Negotiation 설정

* 요청 본문 또는 응답 본문을 어떤 (MIME) 타입으로 보내야 하는지 결정하는 전략을 설정한다.

<br>

## 8. Spring MVC 설정

* Spring MVC 설정이란 DispatcherServlet이 사용할 여러 Bean의 설정을 의미한다.
* HandlerMapper 및 HandlerAdapter 등을 일일히 등록하려니 너무 많고, 해당 빈들이 참조하는 또 다른 객체들까지 설정하려면 비용이 많이 든다.

### 9.1. @EnableWebMvc

* 애노테이션 기반의 Spring MVC 설정을 간편화한다.
  * 해당 어노테이션을 통해 기본 설정이 적용된다.
* WebMvcConfigurer가 제공하는 메소드를 구현하여 커스터마이징할 수 있다.

### 9.2. Spring Boot

* Spring Boot 자동 설정을 통해 다양한 Spring MVC 기능을 아무런 설정 파일을 만들지 않아도 제공한다.
* WebMvcConfigurer가 제공하는 메소드를 구현하여 커스터마이징할 수 있다.
* @EnableWebMvc를 사용하면 Spring Boot 자동 설정을 사용하지 못한다.

### 9.3. Spring MVC 설정 방법

* Spring Boot를 사용하는 경우에는 application.properties 부터 시작한다.
* 한계가 있다면 WebMvcConfigurer로 커스터마이징한다.
* 한계가 있다면 @Bean으로 MVC 구성 요소를 직접 등록한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
