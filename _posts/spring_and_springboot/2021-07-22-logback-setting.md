---
title: "Logging 및 Spring Boot Logback 설정"
excerpt: "Logback 설정을 통해 다양한 형식으로 로그를 작성 및 저장해보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-07-22
last_modified_at: 2021-07-22
---

> [실습 Repository](https://github.com/xlffm3/springboot-logback-practice)

## 1. Logging

로깅이란 시스템이 동작할 때 시스템의 상태 및 동작 정보를 시간 경과에 따라 기록하는 것을 의미한다.

* 개발 과정 혹은 개발 후에 발생할 수 있는 예상치 못한 애플리케이션의 문제를 진달할 수 있고, 다양한 정보를 수집할 수 있다.
* 의미있는 정보들을 적절한 수준으로 수집하는 것이 중요하다.

<br>

## 2. Logging 관련 용어 정리

초기의 Spring은 JCL(Jakarta Commons Logging)을 사용하여 로깅을 구현했다. JCL을 사용하면 기본적인 인터페이스인 Log와 Log 객체 생성을 담당하는 LogFactory만 구현하면 로깅 구현체 교체가 자유롭다.

* SLF4J : Java 진영의 다양한 로깅 프레임워크들에 대한 공용 인터페이스(Facade)다.
  * 공용 인터페이스를 통해 특정 로깅 프레임워크에서 다른 로깅 프레임워크로 쉽게 전환할 수 있다.
  * JCL은 GC가 제대로 작동하지 않는 등의 단점이 존재한다.
    * 문제 해결을 위해 클래스 로더 대신에 컴파일 시점에 구현체를 선택하도록 변경해야 했으며, 이 때 도입되는 것이 SLF4J다.
* Log4j : Apache의 Java 기반의 로깅 프레임워크다.
  * 가장 오래된 로깅 프레임워크며, 현재는 개발이 중단되었다.
* Logback : Log4j 이후에 출시된 로깅 프레임워크로, Spring Boot가 기본적으로 사용한다.
  * Log4j보다 향상된 성능 및 필터링 옵션을 제공한다.
  * 로그 레벨 변경 등에 대해 서버를 재시작할 필요 없이 자동 리로딩을 지원한다.
* Log4j2 : 파일뿐만 아니라 HTTP, DB, Kafka에 로그를 남길 수 있으며 비동기적인 로거를 지원한다.
  * 로깅 성능이 중요시될 때 Log4j2 사용을 고려한다.
  * spring-boot-starter-web 모듈에 Logback을 사용하는 spring-boot-starter-logging 모듈이 포함되어있다.
  * Log4j2 사용을 위해서는 spring-boot-starter-logging 모듈을 exclude하고 spring-boot-starter-logging-log4j2 의존성을 주입해야 한다.
  * Logback과 달리 멀티 쓰레드 환경에서 비동기 로거(Async Logger)의 경우 Log4j 1.x 및 Logback보다 성능이 우수하다.

<br>

## 3. Logging 방법

> ControllerAdvice.java

```java
private static final Logger LOGGGER = LoggerFactory.getLogger(ControllerAdvice.class);
private static final Logger LOGGGER = LoggerFactory.getLogger("file-appender");
```

* ``LoggerFactory.getLogger()`` 메서드 호출을 통해 Logger 객체를 쉽게 받아올 수 있다.
  * 특정 클래스명에 부합하는 Logger를 호출하거나, XML 등에 정의된 특정 Logger를 호출한다.
* Lombok을 사용한다면 @Slf4j 애너테이션을 클래스에 부착함으로써 자동으로 log라는 변수명의 Logger 객체를 받아올 수 있다.
* ``warn()``, ``info()``, ``debug()``, ``error()`` 등 다양한 메서드 호출을 통해 로그를 남길 수 있다.

<br>

## 4. Logback 설정

별도의 설정없이 Logger 객체로 로그를 남기면, 콘솔에 로그가 표준 출력되는 것이 전부다. 다음과 같은 니즈가 있다면 별도의 Logback 설정을 구성해야 한다.

* 특정 경로의 파일에 로그를 작성한다.
* 로그를 날짜 별로 분리한다.
* 특정 기간이 지나면 로그를 삭제한다.
* Profile에 따라 로깅 환경 구성을 분리한다.
* 등등

YAML, XML, Java 등의 파일로 Logback 옵션에 대해 세밀하게 설정할 수 있다. YAML의 경우 로깅 옵션 커스터마이징에 한계가 많아 XML 사용을 추천한다. XML에 비해 환경 변수 지정 등이 어렵다.

### 4.1. Logback Extension

Logback Extension은 특정 Profile별로 로깅 환경을 다르게 구성할 수 있는 등 부수적인 기능을 지원한다. Logback Extension 기능을 사용하려면 무조건 클래스패스의 ``logback-spring.xml``에 Logback 설정을 정의해야 한다. Logback Extension 기능을 사용하지 않는다면 ``logback.xml``에 Logback 설정을 정의해도 된다.

* Spring Boot는 Logback 관련 설정 파일들을 우선 순위를 두고 스캔하는데, 표준 ``logback.xml`` 설정 파일은 너무 일찍 로드되기 때문에 확장 기능을 해당 파일에서 사용하기 어렵다.
  * Spring 공식 문서에서도 로깅 설정 파일에 -spring 접미사를 붙일 것을 권고한다.
* ``logback-access.xml``을 지정하면 API 요청 응답에 대해 로그를 남길 수 있다.

<br>

## 5. Logback 설정 예제

> local-file-logger.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<included>
  <property name="home" value="logs/local/local"/>
  <appender name="local-file-logger" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${home}-file.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>${home}-file-%d{yyyyMMdd}-%i.log</fileNamePattern>
      <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <maxFileSize>15MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>
    <encoder>
      <charset>utf8</charset>
      <Pattern>
        %d{yyyy-MM-dd HH:mm:ss.SSS} %boldGreen(%-5level) %boldMagenta(${PID:-}) %boldCyan(%t) %boldYellow(%class{36}_%M) %boldWhite(L:%L) %gray(%logger{36}) %n %boldRed(     >) %m%n
      </Pattern>
    </encoder>
  </appender>
</included>
```

* classpath를 기준으로 ``/logs/local`` 디렉토리에 ``local-file.log`` 형태로 로그 파일이 기록된다.
  * ``${home}``이 환경 변수라고 생각하면 된다.
* rollingPolicy란 기록되는 log 파일이 특정 기간 혹은 용량에 다다르면 분리 및 백업하도록 지시하는 정책이다.
  * ``local-file.log`` 파일의 용량이 명시한 임계 용량 혹은 기간을 지나면, 명시된 패턴과 같이 자동으로 분리된다.
    * ``local-file-2021-06-21.log``
  * 위 예제는 용량 한계만 정의만 되어 있으나, 기간도 설정할 수 있다.
    * 임계 기간 및 용량에 대해 별도 설정이 없다면 기본 설정값을 따르다.
    * 기본 설정값은 공식 문서를 참고하길 바란다.
* 로그 파일에 기록되는 로그의 형태 또한 지정할 수 있다.
  * 패턴 : [https://goddaehee.tistory.com/206](https://goddaehee.tistory.com/206)
  * 색상 : [https://hue9010.github.io/etc/logback-설정하기/](https://hue9010.github.io/etc/logback-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)
    * janino라는 라이브러리를 추가하면 XML 파일에 if 조건을 명시함으로써, 특정 상황에서 표현할 로그의 색상 및 패턴을 분기 처리할 수 있다.

> logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
  <include resource="org/springframework/boot/logging/logback/base.xml"/>
  <include resource="./logback/prod-file-logger.xml"/>
  <include resource="./logback/local-file-logger.xml"/>
  <include resource="./logback/test-file-logger.xml"/>

  <springProfile name="prod">
    <logger name="com.woowacourse.pickgit" level="INFO">
      <appender-ref ref="prod-file-logger"/>
    </logger>
    <logger name="org.springframework" level="INFO">
      <appender-ref ref="prod-file-logger"/>
    </logger>
    <logger name="org.hibernate.tool.hbm2ddl" level="DEBUG">
      <appender-ref ref="prod-file-logger"/>
    </logger>
  </springProfile>

  <springProfile name="local">
    <logger name="com.woowacourse.pickgit" level="INFO">
      <appender-ref ref="local-file-logger"/>
    </logger>
    <logger name="org.springframework" level="INFO">
      <appender-ref ref="local-file-logger"/>
    </logger>
    <logger name="org.hibernate.SQL" level="DEBUG"> //SQL 로그
      <appender-ref ref="local-file-logger"/>
    </logger>
    <logger name="org.hibernate.tool.hbm2ddl" level="DEBUG"> //DDL 로그
      <appender-ref ref="local-file-logger"/>
    </logger>
    <logger name="org.hibernate.type" level="TRACE"> ////질의에 바인딩되는 파라미터 및 질의 결과 등 다양한 로그
      <appender-ref ref="local-file-logger"/>
    </logger>
    <logger name="org.hibernate.type.BasicTypeRegistry" level="WARN">
      <appender-ref ref="local-file-logger"/>
    </logger>
  </springProfile>

  <springProfile name="test">
    <logger name="com.woowacourse.pickgit" level="INFO">
      <appender-ref ref="test-file-logger"/>
    </logger>
    <logger name="org.springframework" level="INFO">
      <appender-ref ref="test-file-logger"/>
    </logger>
    <logger name="org.hibernate.SQL" level="DEBUG">
      <appender-ref ref="test-file-logger"/>
    </logger>
    <logger name="org.hibernate.tool.hbm2ddl" level="DEBUG">
      <appender-ref ref="test-file-logger"/>
    </logger>
    <logger name="org.hibernate.type" level="TRACE">
      <appender-ref ref="test-file-logger"/>
    </logger>
    <logger name="org.hibernate.type.BasicTypeRegistry" level="WARN">
      <appender-ref ref="test-file-logger"/>
    </logger>
  </springProfile>
</configuration>
```

활성화된 특정 Profile에 따라 로깅 환경을 다르게 설정할 수 있다.

* ``com.woowacourse.pickgit``와 같이 개인이 개발한 프로젝트에 대해 특정 레벨 이상의 로그를 파일로 기록할 수 있다.
* ``org.springframework``와 같이 Spring 프레임워크가 구동될 때 발생하는 로그를 파일로 기록할 수 있다.
* 웹 어플리케이션 작동 중 Hibernate가 발생시키는 SQL 쿼리를 로그로 담고 싶다면 ``org.hibernate``로 시작하는 패키지에 대해 로거를 지정해준다.
  * SQL 쿼리, DDL 쿼리, 파라미터 바인딩 결과, 통계 등 다양한 정보를 로그로 남길 수 있다.
  * 패키지별로 관장하는 로그가 다르니 상세한 내용은 하단 링크를 참고하자.
    * [https://kwonnam.pe.kr/wiki/java/hibernate/log](https://kwonnam.pe.kr/wiki/java/hibernate/log)
    * [https://m.blog.naver.com/kh2un/222008545174](https://m.blog.naver.com/kh2un/222008545174)
  * properties 혹은 YAML에 ``spring.jpa.properties.hibernate.format_sql=true``를 지정하면 로그에 남겨지는 SQL 쿼리의 가독성이 증가한다.
  * ``org.hibernate.type`` 하위의 BasicBinder 및 BasicExtractor 등의 패키지에 로거를 등록하면 질의에 바인딩되는 파라미터 및 질의 결과 등을 로그로 남길 수 있다.

![image](https://user-images.githubusercontent.com/56240505/126643030-f8e94456-2723-4032-b5a3-d8f75ca5d2b4.png)

* 로그 레벨 정보, PID, 스레드, 클래스 및 메서드 명, 라인 넘버, 로거 이름 등의 출력 양식을 커스터마이징한 모습이다.

<br>

---

## References

* [Spring Boot Logging Best Practices Guide](https://coralogix.com/blog/spring-boot-logging-best-practices-guide/)
* [A Guide To Logback](https://www.baeldung.com/logback)
* [Logging in Spring Boot](https://www.baeldung.com/spring-boot-logging)
* [How do I configure Logback to print out the class name](https://stackoverflow.com/questions/15258144/how-do-i-configure-logback-to-print-out-the-class-name/19950046)
* [Log4j2 Logback Log4j 차이 및 적용](https://junshock5.tistory.com/124)
