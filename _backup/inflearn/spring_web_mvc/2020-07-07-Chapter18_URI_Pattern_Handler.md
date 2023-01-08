---
title: "[Spring Web MVC] 18장 : URI Pattern Handler"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-15T08:09:00-05:00
---

## 1. Handler Method

* 핸들러 메소드 아규먼트 : 주로 요청 그 자체 또는 요청에 들어있는 정보를 받아오는데 사용한다.
* 핸들러 메소드 리턴 : 주로 응답 또는 모델을 랜더링할 뷰에 대한 정보를 제공하는데 사용한다.

<br>

## 2. URI Pattern

> SampleController.java

```java
@Controller
public class SampleController {

    @GetMapping("/events/{id}")
    @ResponseBody
    public Event event(@PathVariable("id") Optional<Integer> id) {
        Event event = new Event();
        event.setId(id);
        return event;
    }
}
```

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

* @PathVariable
	* 요청 URI 패턴의 일부를 핸들러 메소드 아규먼트로 받는 방법이다.
	* 타입 변환을 지원한다.
    * 원한다면 Formatter나 Converter를 등록해 기본값이 아닌 특정 객체로 변환 가능하다.
	* (기본)값이 반드시 있어야 한다.
	* 표현식과 매개변수의 이름이 같다면 name 생략이 가능하다.
	* Required 조건을 설정할 수 있다.
	* Optional을 지원한다.
* @MatrixVariable.
	* 요청 URI 패턴에서 키/값 쌍의 데이터를 메소드 아규먼트로 받는 방법이다.
	* 타입 변환을 지원한다.
	* (기본)값이 반드시 있어야 한다.
	* Optional을 지원한다.
	* 이 기능은 기본적으로 비활성화 되어 있다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
