---
title:  "[Spring Boot 개념과 활용] 12장 : 정적 리소스 지원"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:03:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. Static Resource 지원

* 동적으로 View를 생성하는 것이 아니라, 요청이 오기 전부터 이미 준비되어 있는 정적인 자원들을 의미한다.
* 정적 리소스 맵핑 “/\*\*”.
* 기본 리소스 위치.
  * classpath:/static
  * classpath:/public
  * classpath:/resources/
  * classpath:/META-INF/resources
  * spring.mvc.static-path-pattern : 특정 맵핑 설정 변경이 가능하다.
    * 예) “/hello.html” => /static/hello.html
  * spring.mvc.static-locations : 리소스 찾을 위치의 변경이 가능하다.
* Last-Modified 헤더를 보고 304 응답을 보낸다.
* ResourceHttpRequestHandler가 요청을 처리한다.
  * properties에 대한 추가보다 해당 방법이 권장된다.
  * WebMvcConfigurer의 ``addRersourceHandlers()``로 커스터마이징 할 수 있다.

<br>

## 2. addRersourceHandlers

> Configuration.java

```java
@Configuration
public class Config implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/m/**")
                .addResourceLocations("classpath:/m/")
                .setCachePeriod(20);
    }
}
```

* 기존 Spring Boot가 제공하는 리소스 핸들러를 유지하면서 추가한다.
* m으로 시작하는 요청이 오면 classpath 기준 m 디렉토리 하위 파일로 검색한다는 의미이다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
