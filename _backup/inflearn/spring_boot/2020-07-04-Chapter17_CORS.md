---
title:  "[Spring Boot 개념과 활용] 17장 : CORS"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:08:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. SOP와 CORS

* SOP : Single-Origin Policy.
  * Origin이 다른 경우 호출이 불가능하다.
* CORS : Cross-Origin Resource Sharing.
  * SOP를 우회하기 위한 표준 기술이다.
* Origin이란?
  * URI 스키마. (http, https)
  * hostname. (jipark.me, localhost)
  * 포트. (8080, 18080)

<br>

## 2. CORS 예제

> index.html

```html
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<script>
    $(function () {
        $.ajax("http://localhost:8080/hello")
            .done(function (msg) {
                alert(msg);
            })
        .fail(function () {
            alert("fail");
        });
    })
</script>
```

> UserController.java

```java
@CrossOrigin(origins = "http://localhost:18080")
@GetMapping("/hello")
public String hello() {
    return "hello";
}
```

* 18080 포트의 클라이언트가 8080 포트에 성공적으로 접근한 경우, message가 출력이 되어야 한다.
* 8080 포트의 서버는 해당 메서드에 CORS를 허용하는 어노테이션을 붙인다.
  * 해당 어노테이션은 메서드 단위, 혹은 클래스 단위로 붙일 수 있다.

<br>

## 3. CORS 글로벌 설정

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:18080");
    }
}
```

* MVC 확장 기능을 통해 Configuration 클래스에 CorsMappings을 추가해준다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
