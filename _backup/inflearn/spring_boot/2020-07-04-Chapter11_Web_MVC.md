---
title:  "[Spring Boot 개념과 활용] 11장 : Spring Web MVC"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:02:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. Spring Web MVC

* Spring Boot는 @EnableAutoConfiguration을 통해 Web MVC에 필요한 기본 기능들을 자동으로 설정해준다.
* Spring MVC 확장(권장)
  * 별도의 @Configuration 클래스 파일을 생성하고 ``implements WebMvcConfigurer`` 구현을 통해, 별도의 기능을 추가로 커스터마이징한다.
* Spring MVC 재정의
  * @Configuration + @EnableWebMVC
  * @EnableWebMVC 애너테이션은 Spring Boot의 자동 설정 기능을 사용하지 못하기 때문에 권장하지 않는다.

<br>

## 2. HttpMessageConverters

> UserController.java

```java
@RestController
public class UserController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @PostMapping("/users/create")
    public @ResponseBody User create(@RequestBody User user) {
        return user;
    }
}
```

> UserControllerTest.java

```java
@Test
public void createUser_JSON() throws Exception {
    String userJson = "{\"username\":\"jipark\", \"password\":\"123\"}";

    mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON)
            .accept(MediaType.APPLICATION_JSON)
            .content(userJson))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username", is(equalTo("jipark"))))
            .andExpect(jsonPath("$.password", is(equalTo("123"))));
}
```

* HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP 응답 본문으로 변경할 때 사용한다.
* RestController는 @ResponseBody를 생략할 수 있다.
  * 일반 Controller는 Return 값에 해당하는 View Name을 Resolve하려고 한다.
  * RestController는 적합한 메시지 컨버터를 사용해서 응답 본문을 담는다.

<br>

## 3. ViewResolver

> pom.xml

```xml
<dependency>
  <groupId>"com.fasterxml.jackson.dataformat"</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
  <version>2.9.6</version>
</dependency>
```

> UserControllerTest.java

```java
@Test
public void createUser_XML() throws Exception {
    String userJson = "{\"username\":\"jipark\", \"password\":\"123\"}";

    mockMvc.perform(post("/users/create")
            .contentType(MediaType.APPLICATION_JSON)
            .accept(MediaType.APPLICATION_XML)
            .content(userJson))
            .andExpect(status().isOk())
            .andExpect(xpath("/User/username").string("jipark"))
            .andExpect(xpath("/User/password").string("123"));
}
```

* 들어오는 요청의 accept header에 따라 응답이 달라진다.
  * 기본적인 JSON 메시지 컨버터 등은 스프링 부트에서 제공한다.
* 요청의 응답을 찾을 수 있는 View를 찾아 리턴하는데, 위 예제는 XML 메시지 컨버터를 추가하여 사용하는 것이다.
* 미디어 타입이 XML인 메시지를 컨버팅 할 수 있는 HttpMessageConverter가 있어야 테스트를 통과할 수 있다.

<br>

## 4. @MVC 예외 처리 방법

> Controller.java

```java
@GetMapping("/exception")
public String exception() {
    throw new SampleException();
}

@ExceptionHandler(SampleException.class)
public @ResponseBody AppError sampleError(SampleException e) {
    AppError appError = new AppError();
    appError.setMessage("aa");
    appError.setReason("bb");
    return appError;
}
```

* hello 요청이 들어오면, @ExceptionHandler로 해당 예외를 catch하여 커스텀한 메시지 응답을 보낼 수 있다.
* 단, 해당 방식은 정의되어 있는 컨트롤러 내부에서만 사용이 가능하다.
* ExceptionHandler를 전역적으로 사용하는 방법.
  * 별도의 클래스에 @ControllerAdvice를 붙인다.
  * 해당 클래스 내부에 ExceptionHandler를 정의한다.

<br>

## 5. Spring Boot가 제공하는 기본 예외 처리기

* BasicErrorController.
  * HTML과 JSON 응답을 지원한다.
* 커스터마이징 방법.
  * ErrorController 인터페이스를 구현한다.

<br>

## 6. 커스텀 예외 페이지

* 상태 코드 값에 따라 정적인 에러 페이지를 보여줄 수 있다.
* 예외 핸들러가 없으면 해당 상태 코드에 부합하는 에러 페이지를 맵핑해 보여준다.
  * ``src/main/resources/static|template/error/``
  * 404.html
  * 5xx.html
* ErrorViewResolver 구현하기.
  * 조금 더 동적인 정보를 담는 에러 페이지를 보여줄 수 있는 방법이다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
