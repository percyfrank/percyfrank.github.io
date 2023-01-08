---
title: "[Spring Web MVC] 8장 : HandlerInterceptor"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T08:08:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. HandlerInterceptor

```java
//preHandle1
//preHandle2
//요청 처리
//postHandle2
//postHandle1
//뷰 렌더링
//afterCompletion2
//afterCompletion1
```

* 핸들러 맵핑에 설정할 수 있는 인터셉터이다.
* 핸들러를 실행하기 전, 후(아직 랜더링 전) 그리고 완료(랜더링까지 끝난 이후) 시점에 부가 작업을 하고 싶은 경우에 사용할 수 있다.
* 여러 핸들러에서 반복적으로 사용하는 코드를 줄이고 싶을 때 사용할 수 있다.
	* 로깅, 인증 체크, Locale 변경 등.

<br>

## 2. boolean preHandle(request, response, handler)

* 핸들러 실행하기 전에 호출된다.
* “핸들러"에 대한 정보를 사용할 수 있기 때문에 서블릿 필터에 비해 보다 세밀한 로직을 구현할 수 있다.
* 리턴값으로 계속 다음 인터셉터 또는 핸들러로 요청 및 응답을 전달할지(true) 혹은 응답 처리가 이곳에서 끝났는지(false) 알린다.

<br>

## 3. void postHandle(request, response, modelAndView)

* 핸들러 실행이 끝나고 아직 뷰를 랜더링 하기 이전에 호출된다.
* “뷰"에 전달할 추가적이거나 여러 핸들러에 공통적인 모델 정보를 담는데 사용할 수도 있다.
* 이 메소드는 인터셉터 역순으로 호출된다.
* 비동기적인 요청 처리 시에는 호출되지 않는다.

<br>

## 4. void afterCompletion(request, response, handler, ex)

* 요청 처리가 완전히 끝난 뒤(뷰 랜더링 끝난 뒤)에 호출된다.
* preHandler에서 true를 리턴한 경우에만 호출된다.
* 이 메소드는 인터셉터 역순으로 호출된다.
* 비동기적인 요청 처리 시에는 호출되지 않는다.

<br>

## 5. vs 서블릿 필터

* 서블릿 보다 구체적인 처리가 가능하다.
* 서블릿은 보다 일반적인 용도의 기능을 구현하는데 사용하는게 좋다.

<br>

## 6. HandlerInterceptor 등록 및 구현

> GreetingInterceptor.java

```java
public class GreetingInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("preHandle 1");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle 1");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion 1");
    }
}
```

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new GreetingInterceptor()).order(0);
        registry.addInterceptor(new AnotherInterceptor())
                .addPathPatterns("/hi/**")
                .order(-1);
    }
}
```

* 특정 패턴에 해당하는 요청에만 적용할 수도 있다.
* 순서를 지정할 수 있다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
