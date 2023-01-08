---
title:  "[Spring Boot 개념과 활용] 7장 : Profile"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:09:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. Profile

* @Profile의 위치
  * @Configuration.
  * @Component.
  * @ConfigurationProperties
* 어떤 프로파일을 활성화 할 것인가?
  * spring.profiles.active=XXX 옵션을 추가하면 된다.
  * application.properties에 추가할 수 있으며, properties 우선순위에 영향을 받는다.
  * 즉, 커맨드 라인(-Dspring.profiles.active=???)이나 VM 옵션 등으로 사용할 수 있다.
* 프로파일용 프로퍼티.
  * application-{profile}.properties=.
  * 일반 application.properties보다 우선순위가 높다.
  * active 시키는 프로파일 이름에 해당하는 properties를 참조한다.
* 어떤 프로파일을 추가할 것인가?
  * spring.profiles.include=XXX.
  * 현재 참조 중인 properties에서 다른 profile을 추가할 경우, 해당 프로파일에 해당하는 프로퍼티 파일 내용을 참조할 수 있다.
* **특정 프로파일을 활성화시켜도 기본 프로파일은 항상 활성화된 상태이다.**

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
