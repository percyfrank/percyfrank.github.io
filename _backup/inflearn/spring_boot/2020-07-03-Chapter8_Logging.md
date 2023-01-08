---
title:  "[Spring Boot 개념과 활용] 8장 : Logging"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:10:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. 로깅 퍼사드 vs 로거

* Spring Boot는 Commons Logging이라는 로거 API를 추상화 한 인터페이스를 사용한다.
  * SLF4j, JUL, Log4J2, Logback 등에 대한 기본 설정을 제공한다.
* Spring-JCL.
  * Commons Logging을 컴파일 시점에 SLF4J나 Log4J2로 변경한다.
  * pom.xml에 exclusion을 시키지 않아도 된다.
  * 최종적으로 Logback(SLF4j의 구현체)으로 찍히게 된다.

<br>

## 2. Spring Boot Logging

> application.properties

```properties
logging.path=logs
logging.level.me.glenn.test=DEBUG
```

> AppRunner.java

```java
private Logger logger = LoggerFactory.getLogger(AppRunner.class);
```

* 기본 포맷
  * VM option 혹은 application.properties에 옵션을 추가한다.
* --debug (일부 핵심 라이브러리만 디버깅 모드로 쓴다.)
* --trace (전부 다 디버깅 모드로 쓴다.)
* 컬러 출력 : spring.output.ansi.enabled
* 파일 출력 : logging.file 또는 logging.path
  * 전자는 고정 path에 특정 파일을 생성하고, 후자는 특정 path에 spring.log 파일을 생성한다.
* 로그 레벨 조정 : logging.level.패지키 = 로그 레벨.

<br>

## 3. 커스텀 로그 설정

* SpringBoot의 로깅 시스템을 강제로 변경할 수 있다.
  * 로깅 시스템은 ApplicationContext가 생성되기 이전에 초기화되기 때문에, @PropertySources 등으로 통제할 수 없다.
  * ``LoggingSystem``이라는 System properties를 통해 설정해야 한다.
* Logback(권장) : logback-spring.xml
* Log4J2 : log4j2-spring.xml
* JUL(권장x) : logging.properties
* Logback extension.
  * 프로파일 : \<springProfile name=”프로파일”\>
  * Environment 프로퍼티 : \<springProperty\>
  * [레퍼런스 참조](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html)

<br>

## 4. 로거를 Log4J2로 변경하기

* 웹 서버 변경과 동일한 방식을 적용하면 된다.
* [레퍼런스 참조](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging)

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
