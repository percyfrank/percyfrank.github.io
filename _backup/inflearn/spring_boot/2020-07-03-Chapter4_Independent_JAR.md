---
title:  "[Spring Boot 개념과 활용] 4장 : 독립적으로 실행 가능한 JAR"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:06:00-05:00
---

## 1. JAR

* MAVEN 혹은 GRADLE 프로젝트를 JAR 파일로 패키징할 수 있다.
  * mvn package를 하면 실행 가능한 JAR 파일 “하나가" 생성된다.
  * 이는 spring-maven-plugin이 해주는 패키징 작업이다.
  * 압축을 해제해보면 Jar 파일 내부에 어플리케이션 파일과 의존성 Jar들이 내장되어 있다.

### 1.1. Nested JARs

* Java는 중첩된 내장 Jar 파일을 로드하는 표준적인 방법을 제시하지 않는다.
  * 중첩된 내장 Jar란 Jar 파일 내부에 다른 Jar를 포함하고 있는 것이다.
* 이러한 문제 해결을 위해, 과거에는 “uber” Jar 를 사용했다.
  * 모든 클래스(의존성 및 애플리케이션)를 하나로 압축하는 방법이다.
  * 무슨 라이브러리를 사용하는지, 해당 클래스가 어디 소속인지 알기 어렵다.
  * 또한 내용은 다르지만 이름이 같은 파일의 경우 처리하기 어렵다.

<br>

## 2. Spring Boot의 전략

* 애플리케이션 클래스와 라이브러리 위치를 구분한다.
* ``org.springframework.boot.loader.jar.JarFile``을 사용해서 내장 JAR를 읽는다.
* ``org.springframework.boot.loader.Launcher``를 사용해서 실행한다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
