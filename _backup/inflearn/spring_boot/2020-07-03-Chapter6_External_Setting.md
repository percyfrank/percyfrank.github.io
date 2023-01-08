---
title:  "[Spring Boot 개념과 활용] 6장 : 외부 설정"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:08:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. 사용할 수 있는 외부 설정

* properties
* YAML
* 환경 변수
* 커맨드 라인 아규먼트

<br>

## 2. Property 우선 순위

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties.
2. @PropertySource.
3. application.properties와 같은 설정 파일.  
  * JAR 안에 있는 기본 application properties.
  * JAR 안에 있는 특정 프로파일용 application properties.
  * JAR 밖에 있는 application properties.
  * JAR 밖에 있는 특정 프로파일용 application properties.
4. RandomValuePropertySource.
5. OS 환경 변수
6. ``System.getProperties()`` 자바 시스템 프로퍼티.
7. java:comp/env JNDI 애트리뷰트.
8. ServletContext 파라미터.
9. ServletConfig 파라미터.
10. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로티) 에 들어있는 프로퍼티.
11. 커맨드 라인 인자.
12. test에 있는 properties 특성.
13. @TestPropertySource.

<br>

## 3. application.properties 우선 순위

* 높은게 낮은걸 덮어 쓴다.

1. classpath 루트
2. classpath의 /config 패키지
3. 현재 디렉토리
4. 현재 디렉토리의 하위 /config 패키지
5. 하위 /config 패키지의 하위 디렉토리.

<br>

## 4. 랜덤값 설정하기

> application.properties

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number-less-than-ten=${random.int(10)}
my.number-in-range=${random.int[1024,65536]}
```

<br>

## 5. 플레이스 홀더

> application.properties

```properties
name=jinhong
fullName=${name} park.
```

<br>

## 6. Type-Safe @ConfigurationProperties

> application.properties

```properties
jinhong.name=jinhong
jinhong.age=13
jinhong.fullName=${name} park
jinhong.sessionTimeout=25s
```

> JinhongProperties.java

```java
@Getter @Setter
@Component
@ConfigurationProperties("jinhong")
@Validated
public class JinhongProperties {

    @NotEmpty
    private String name;
    private int age;
    private String fullName;

    @DurationUnit(ChronoUnit.SECONDS) //생략 가능
    private Duration sessionTimeout = Duration.ofSeconds(30);
    //property가 비어있다면 기본값 30
}
```

> Application.java

```java
@SpringBootApplication
@EnableConfigurationProperties(JinhongProperties.class)
public class Application {

    @Autowired
    JinhongProperties jinhongProperties;

    //Type-Safe하지 않은 방식
    @Value("${jinhong.age}")
    int age;
}
```

* EnableConfigurationProperties는 자동으로 등록되기 때문에 생략해도 된다.
* Properties 관련 클래스를 Component로 지정하여, Autowired를 받은 다음 getter를 이용한다.
  * AutoConfiguration 내용과 동일하다.
  * 다른 Bean들에게 주입이 가능하다.

### 6.1. 장점

* 다양한 바인딩 방식을 제공한다.
  * context-path (케밥).
  * context_path (언드스코어).
  * contextPath (캐멀).
  * CONTEXTPATH.
* 기존 @Value와 같은 annotation은 properties를 정확하게 명시해야 했다.
  * jinhong.full-name이라면 ${jinhong.full-name}.
* 그러나 @ConfigurationProperties는 여러 표기법을 융통성 있게 바인딩할 수 있다.
  * @DurationUnit 등을 통해 타입 컨버젼을 지원한다.
  * @Validated과 여러 어노테이션 혹은 @NonNull 등을 통해 값을 검증할 수 있다.
* @Value 등은 SpEL을 사용할 수 있으나 위의 기능들을 사용하지 못한다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
