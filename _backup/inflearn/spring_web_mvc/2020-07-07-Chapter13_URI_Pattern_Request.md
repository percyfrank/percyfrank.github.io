---
title: "[Spring Web MVC] 13장 : URI Pattern Request"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot  
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-15T08:04:00-05:00
---

## 1. 요청 식별자로 맵핑하기

* @RequestMapping은 다음의 패턴을 지원한다.
	* ? : 한 글자 (“/author/???” => “/author/123”).
	* \* : 여러 글자 (“/author/\*” => “/author/keesun”).
	* \*\* : 여러 패스 (“/author/\*\* => “/author/keesun/book”).

<br>

## 2. 클래스에 선언한 @RequestMapping과 조합

> SampleController.java

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @RequestMapping("/**")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
```

* 클래스에 선언한 URI 패턴뒤에 이어 붙여서 맵핑한다.

<br>

## 3. 정규 표현식

* /{name:정규식}.

<br>

## 4. 패턴의 중복

* 가장 구체적으로 맵핑되는 핸들러를 선택한다.

<br>

## 5. URI 확장자 맵핑 지원

* Spring MVC는 @RequestMapping({"/jipark", "/jipark.\*"})을 자동으로 지원한다.
  * jipark.json, jipark.xml 등.
* 이 기능은 권장하지 않고, Spring Boot는 기본으로 이 기능을 사용하지 않도록 설정한다.
	* 보안 이슈 (RFD Attack).
	* URI 변수, Path 매개변수, URI 인코딩 등을 사용할 때 할 때 불명확하다.
* XML, HTML 등 특정 타입을 원한다면 헤더에 해당 정보를 입력하거나, `?type=xml` 과 같이 요청 파라미터를 이용하는 것이 권장된다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
