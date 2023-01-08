---
title: "[Spring Web MVC] 5장 : Spring MVC 구성 요소"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:05:00-05:00
---

## 1. Spring MVC Components

![DispatcherServlet](https://user-images.githubusercontent.com/56240505/80305103-c29c9380-87f5-11ea-8a60-1ae157c2fe84.png)

<br>

## 2. DispatcherSerlvet의 기본 전략

* DispatcherServlet.properties.

### 2.1. MultipartResolver

* 파일 업로드 요청 처리에 필요한 인터페이스이다.
* HttpServletRequest를 MultipartHttpServletRequest로 변환해주어 요청이 담고 있는 File을 꺼낼 수 있는 API 제공한다.

### 2.2. LocaleResolver

* 클라이언트의 위치(Locale) 정보를 파악하는 인터페이스이다.
* 기본 전략은 요청의 accept-language를 보고 판단한다.

### 2.3. ThemeResolver

* 애플리케이션에 설정된 테마를 파악하고 변경할 수 있는 인터페이스이다.

### 2.4. HandlerMapping

* 요청을 처리할 핸들러를 찾는 인터페이스이다.

### 2.5. HandlerAdapter

* HandlerMapping이 찾아낸 “핸들러”를 처리하는 인터페이스이다.
* 스프링 MVC 확장력의 핵심이다.

### 2.6. HandlerExceptionResolver

* 요청 처리 중에 발생한 에러 처리하는 인터페이스이다.
* 대표적으로 @ExceptionHandler Annotation이 있다.

### 2.7. RequestToViewNameTranslator

* 핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우, 요청을 기반으로 뷰 이름을 판단하는 인터페이스이다.

### 2.8. ViewResolver

* 뷰 이름(string)에 해당하는 뷰를 찾아내는 인터페이스이다.

### 2.9. FlashMapManager

* FlashMap 인스턴스를 가져오고 저장하는 인터페이스이다.
* FlashMap은 주로 리다이렉션을 사용할 때 요청 매개변수를 사용하지 않고 데이터를 전달하고 정리할 때 사용한다.
* redirect:/events..

<br>

## 3. Spring MVC의 동작 원리

> DispatcherServlet.java

```java
protected void initStrategies(ApplicationContext context) {
    this.initMultipartResolver(context);
    this.initLocaleResolver(context);
    this.initThemeResolver(context);
    this.initHandlerMappings(context);
    this.initHandlerAdapters(context);
    this.initHandlerExceptionResolvers(context);
    this.initRequestToViewNameTranslator(context);
    this.initViewResolvers(context);
    this.initFlashMapManager(context);
}
```

* DispatcherServlet : 굉장히 복잡한 서블릿이다.
* DispatcherServlet의 초기화.
	1.	특정 타입에 해당하는 Bean을 찾는다.
	2.	없으면 DispatcherServlet.properties 등 기본 전략을 사용한다.

<br>

## 4. Spring Boot를 사용하지 않는 Spring MVC

> WebApplication.java

```java
public class WebApplication implements WebApplicationInitializer {

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

* 톰캣 등 서블릿 컨네이너에 등록한 웹 애플리케이션(WAR)에 DispatcherServlet을 등록한다.
	* web.xml에 서블릿을 등록한다.
	* (Spring 3.1+, 서블릿 3.0+)또는 WebApplicationInitializer에 Java 코드로 서블릿을 등록한다.
* 세부 구성 요소는 @Configuration의 Bean을 설정하기 나름이다.

<br>

## 5. Spring Boot를 사용하는 Spring MVC

* Java 애플리케이션에 내장 톰캣을 만들고 그 안에 DispatcherServlet을 등록한다.
* Spring Boot 자동 설정이 이를 자동으로 설정해준다.
* Spring Boot의 주관에 따라 여러 인터페이스 구현체를 Bean으로 등록한다.
* 따라서 DispatcherServlet에서 기본 null이었던 MultipartResolver가 자동으로 등록되어, Spring Boot에서는 별다른 설정없이 파일 업로드가 가능하는 등의 이점이 존재한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
