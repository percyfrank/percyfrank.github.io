---
title:  "[Spring Boot 개념과 활용] 13장 : Web Jar"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:04:00-05:00
---

## 1. Web Jar

> hello.html

```html
<script src="/webjars/jquery/dist/3.3.1/jquery.min.js"></script>
<script>
  $(function() {
  console.log("ready!");
  });
</script>
```

* JQuery 등 클라이언트 라이브러리도 Jar를 통해 추가할 수 있다.
* 웹JAR 맵핑 “/webjars/\*\*”
* JQuery의 경우, pom.xml에 의존성을 추가하고 html에 Web Jar를 맵핑한다.
* 버전을 생략하려면 webjars-locator-core 의존성을 추가해준다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
