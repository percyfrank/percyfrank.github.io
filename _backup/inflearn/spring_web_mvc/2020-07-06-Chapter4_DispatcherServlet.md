---
title: "[Spring Web MVC] 4장 : DispatcherServlet"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot  
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:04:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. DispatcherServlet 초기화

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

* Strategy Pattern을 사용하며, 세부 인터페이스들을 초기화한다.
  * 세부 사항의 타입에 해당하는 Bean들을 탐색한다.
  * 사용자가 WebMvcConfigurer 커스터마이징 구현 등을 통해 해당 타입의 Bean들을 추가한 상태라면, 이를 목록에 추가한다.
  * 없다면 기본 전략에 해당되는 Bean들을 등록한다.
    * ``{spring-framework-main-code}/spring-webmvc/src/main/resources/org/springframework/web/servlet/DispatcherServlet.properties[DispatcherServlet.properties]``
* HandlerMapping : 핸들러를 찾아주는 인터페이스.
* HandlerAdapter : 핸들러를 실행하는 인터페이스.
* HandlerExceptionResolver.
* ViewResolver.
* ...

<br>

## 2. DispatcherServlet 동작 순서

0.  WebApplicationContext가 탐색되어 요청의 attribute로 바인딩되며, 컨트롤러 등 다른 요소들이 사용할 수 있는 상태가 된다.
  * 기본적으로 ``DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE``의 키로 바인딩된다.
1.	(로케일, 테마, 멀티파트 등) 요청을 분석한다.
2.	(핸들러 맵핑에게 위임하여) 요청을 처리할 핸들러를 찾는다.
  * 기본적으로 2개의 핸들러 맵핑이 등록되어 있다.
3.	(등록되어 있는 핸들러 어댑터 중에) 해당 핸들러를 실행할 수 있는 “핸들러 어댑터”를 찾는다.
4.	찾아낸 “핸들러 어댑터”를 사용해서 핸들러의 응답을 처리한다.
	* 핸들러의 리턴값을 보고 어떻게 처리할지 판단한다.
		* 뷰 이름에 해당하는 뷰를 찾아서 모델 데이터를 랜더링한다.
		* @ResponseEntity가 있다면 Converter를 사용해서 응답 본문을 만든다.
5.	(부가적으로) 예외가 발생했다면, 예외 처리 핸들러에 요청 처리를 위임한다.
6.	최종적으로 응답을 보낸다.

<br>

## 3. @ResponseEntity가 있는 경우

* RestController는 일반적인 Controller의 Mapping 메소드에 @ResponseBody Annotation이 추가된 Controller이다.
* 이 경우 ModelAndView는 null이다.
* HandlerMapping : RequestMappingHandlerMapping.
* HandlerAdapter : RequestMappingHandlerAdapter.

<br>

## 4. @ResponseEntity가 없고 View가 있는 경우

> SimpleController.java

```java
@org.springframework.stereotype.Controller("/simple")
public class SimpleController implements Controller {

      @Override
      public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse
      response) throws Exception {
            return new ModelAndView("/WEB-INF/simple.jsp");
      }
}
```

* HandlerMapping : BeanNameUrlHandlerMapping.
* HandlerAdapter : SimpleControllerHandlerAdapter.
  * Handler는 SimpleController 클래스이다.
* RequestMappingHandlerAdapter는 Annotation 기반의 Spring MVC 핸들러를 실행하며, SimpleControllerHandlerAdapter가 Controller 인터페이스를 구현한 핸들러를 실행할 수 있다.
* @ResponseBody가 없는 경우이다.
  * 리턴된 뷰 네임을 기반으로 ModelAndView 객체를 생성하고, InternalResourceViewResolver를 통해 뷰 네임에 해당하는 실제 리소스를 찾는다.
  * 이후 Model 객체를 바인딩하여 View를 렌더링한다.
  * 렌더링의 의미는 만들어놓은 JSP 등 리소스를 Response에 실어서 보낸다는 의미이다.

<br>

## 5. Custom ViewResolver

> WebConfig.java

```java
@Configuration
@ComponentScan
public class WebConfig {

    @Bean
    public InternalResourceViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

> SimpleController.java

```java
@org.springframework.stereotype.Controller("/simple")
public class SimpleController implements Controller {

      @Override
      public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse
      response) throws Exception {
            return new ModelAndView("simple");
      }
}
```

* ViewResolver 및 HandlerMapping 등을 별도로 등록해놓으면 Custom한 Bean들을 사용하며, 없을 경우 기본 전략으로 설정된 기능들을 이용한다.
* 위 예제는 ViewResolver에서 suffix와 prefix를 정의한 Custom ViewResolver이다.
* ViewResolver : InternalResourceViewResolver.
* InternalResourceViewResolver : Prefix & Suffix.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
