---
title:  "[Spring Boot 개념과 활용] 10장 : Devtools"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:01:00-05:00
---

## 1. Spring Boot Devtools

* 레퍼런스를 참고하여 의존성을 주입한다.
* 캐시 설정을 개발 환경에 맞게 변경해준다.
  * 클래스패스에 있는 파일이 변경 될 때마다 자동으로 재시작된다.
  * 직접 껐다 키는(cold starts) 것보다 빠른다.
  * JRebel같은 것은 아니며, 릴로딩보다는 느리다.
  * 리스타트 하고 싶지 않은 리소스에 대한 설정.
    * ``spring.devtools.restart.exclude``
  * 리스타트 기능 끄기.
    * ``spring.devtools.restart.enabled = false``
* 라이브 릴로드
  * 리스타트 했을 때 브라우저를 자동 리프레시 하는 기능이다.
  * 브라우저 플러그인을 설치해야 한다.
  * 라이브 릴로드 서버 끄기.
    * ``spring.devtools.liveload.enabled = false``
* 글로벌 설정.
  * ``~/.spring-boot-devtools.properties``
* 리모트 애플리케이션.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
