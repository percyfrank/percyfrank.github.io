---
title: "[Spring Web MVC] 16장 : Head & Options"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-15T08:07:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. Spring Web MVC에서 자동으로 처리하는 HTTP Method

> SimpleControllerTest.java

```java
@Test
public void hello() throws Exception {
    this.mockMvc.perform(options("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"))
                .andExpect(header().stringValues(HttpHeaders.ALLOW, "GET", "POST", "HEAD", "OPTIONS"));
}
```

*	HEAD.
* OPTIONS.

<br>

## 2. HEAD

* GET 요청과 동일하지만 응답 본문을 받아오지 않고 응답 헤더만 받아온다.

<br>

## 3. OPTIONS

* 사용할 수 있는 HTTP Method 목록을 제공한다.
* 서버 또는 특정 리소스가 제공하는 기능을 확인할 수 있다.
* 서버는 Allow 응답 헤더에 사용할 수 있는 HTTP Method 목록을 제공해야 한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
