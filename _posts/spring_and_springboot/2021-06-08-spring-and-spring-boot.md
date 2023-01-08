---
title: "Spring vs Spring Boot"
excerpt: "Spring은 엔터프라이즈급 어플리케이션 제작을 위한 프레임워크다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-08
last_modified_at: 2021-06-08
---

## 1. Spring 생태계

Spring은 Spring Framework를 기반으로 여러 서브 프로젝트들의 모음(Spring Boot, Spring Data, Spring Security, Spring Batch 등)으로 구성되어 있다.

그리고 각 프로젝트에는 여러 하위 프로젝트(모듈)이 존재한다. 대표적으로 Spring Framework에는 Spring Web MVC, Spring JDBC, Spring Web, Spring Core 등의 하위 모듈이 존재한다.

<br>

## 2. Spring Framework

Spring Framework란 엔터프라이즈급 어플리케이션 제작을 위한 프레임워크다. 특히 객체 지향의 특징을 잘 활용할 수 있게 해주며, 개발자들은 핵심 비즈니스 로직 구현에만 집중하도록 한다.

IT 시스템의 의존도가 높아지면서 어플리케이션의 비즈니스 로직이 복잡해짐과 동시에, ``트랜잭션, 상태관리, 멀티 쓰레딩, 리소스 풀링, 보안`` 등 여러 로우 레벨의 기술적인 처리가 요구되었다. 많은 사용자의 처리 요구를 빠르고 안정적이면서 확장 가능한 형태로 유지하기 위해서이다. 문제는 어플리케이션 개발자가 이 모든 영역을 전문적으로 커버하기란 어려웠다.

Spring의 기본 전략은 비즈니스 로직 코드를 엔터프라이즈 기술 처리 코드로부터 분리하는 것이다. 초기 Spring 기본 설정만 잘 해놓는다면, 추후 엔터프라이즈 기술(Spring 관련 코드)를 크게 신경쓸 일이 없다. 엔터프라이즈 기술이란 분산처리와 트랜잭션 및 보안 등 엔터프라이즈급 어플리케이션이 필요로 하는 기술들을 의미한다. 개발자는 DI, IoC를 통해 객체지향을 극대화하여 비즈니스 로직 구현에 집중하면 된다.

<br>

## 3. Spring Boot

Spring이 발전함에 따라 개발자가 관리해야할 설정이 많아지고 복잡해졌다. Spring Boot는 약간의 설정만으로 Spring 어플리케이션을 쉽게 만들 수 있게 해준다. Spring Framework 사용을 간소화해주는 도구일 뿐, Spring Framework와 별개로 사용할 수 있는 것은 아니다.

### 3.1. 의존성 및 버전 관리

* 기존 Spring은 개발에 필요한 모듈 의존성을 각각 다운받아야 했으며, 각 모듈의 버전을 개발자가 명시해야 했다.
  * Spring Boot는 ``spring-boot-starter``를 지원하며, 자주 사용하는 모듈간의 의존성 모음을 제공해준다.
  * ``spring-boot-starter-parent``를 통해 각 모듈의 현재 Spring Boot 버전에 가장 적잡한 버전을 제공해준다.
  * 따라서 개발자는 별도로 버전을 명시할 필요가 없다.

### 3.2. 자동 설정

@SpringBootApplication 애너테이션 내부의 @SpringBootConfiguration, @EnableAutoConfiguration, @ComponentScan 애너테이션을 통해 자동 설정이 가능하다.

* @SpringBootConfiguration은 Spring Boot 전용 애너테이션이다.
  * @SpringBootTest, @WebMvcTest 등 Spring이 제공하는 테스트는 해당 애너테이션이 붙은 클래스가 없다면 실패한다.
* Classpath에 라이브러리 jar 파일이 등록되면, @EnableAutoConfiguration 애너테이션에 의해 ``spring.factories``의 후보 클래스들을 조건에 따라 Bean으로 등록한다.
* ``spring-configuration-metadata``는 자동 설정에 사용할 프로퍼티 정의 파일로, properties 혹은 yml에 작성한 값을 프로퍼티로 세팅한 후, 구현되어 있는 자동 설정에 값을 주입한다.
* 개발자가 작성한 프로젝트의 Bean들을 스캔해 등록한다.

### 3.3. 내장 WAS

Spring을 통해 웹 어플리케이션을 개발하고 배포를 하려면

1. 어플리케이션을 WAR 패키징한다.
2. WAS를 설치한다.
3. WAS에 WAR 파일을 올린다.

와 같은 과정을 거친다. 반면, Spring Boot는 WAS가 내장되어 있다. 따라서 웹 어플리케이션을 jar 파일로 빌드하여 빠르고 독립적으로 실행할 수 있다.

### 3.4. 모니터링

Spring Boot의 모듈 중 하나인 Actuator는 어플리케이션의 관리 및 모니터링을 지원해준다.

Actuator는 민감한 정보를 제공하기 때문에 운영 시에는 Spring Security로 보안을 신경써야 한다. 또한 데이터는 메모리에 저장되기 때문에 Tomcat을 재시작하면 영속되지 않아 별도로 저장해야 한다.

또한 Actuator는 상용 서비스 수준에서 필요로 할 모니터링 기능을 엔드포인트로 미리 만들어서 제공한다.

* End Point
  * ``/health`` : 어플리케이션의 상태 정보.
  * ``/metrics/{name}`` : 어플리케이션의 CPU 사용량 등 metric 정보.
  * ``/beans`` : 등록된 Bean 정보.
  * ``/loggers/{name}`` : 어플리케이션의 로거 구성.

<br>

---

## References

* [[10분 테코톡] 🏀 에어의 Spring vs Spring Boot](https://www.youtube.com/watch?v=Y11h-NUmNXI)
