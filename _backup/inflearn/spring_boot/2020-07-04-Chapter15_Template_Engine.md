---
title:  "[Spring Boot 개념과 활용] 15장 : Template Engine과 Html Unit"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:06:00-05:00
---

## 1. Template Engine

* Spring Boot가 지원하는 템플릿 엔진들.
  * FreeMarker.
  * Groovy.
  * Thymeleaf.
  * Mustache.
* JSP를 권장하지 않는 이유.
  * JAR 패키징 할 때는 동작하지 않고, WAR 패키징을 해야 한다.
  * Undertow는 JSP를 지원하지 않는다.
* 템플릿 엔진 의존성 추가 및 사용 방법에 대한 내용은 생략한다.

<br>

## 2. Html Unit

> pom.xml

```xml
<dependency>
  <groupId>org.seleniumhq.selenium</groupId>
  <artifactId>htmlunit-driver</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>net.sourceforge.htmlunit</groupId>
  <artifactId>htmlunit</artifactId>
  <scope>test</scope>
</dependency>
```

* Html 템플릿에 대한 단위 테스트가 가능한 라이브러리이다.
* form submission 및 특정 브라우저 Mock을 통한 테스트가 가능하다.
* @Autowire WebClient.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
