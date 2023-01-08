---
title:  "[Spring Boot 개념과 활용] 2장 : 자동 설정"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:04:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. @SpringBootApplication

* @SpringBootApplication을 구성하는 내부의 주요 Annotation은 다음과 같다.
  * @SpringBootConfiguration
  * @ComponentScan
  * @EnableAutoConfiguration
* Bean들은 두 단계로 나눠서 읽힌다.
  * 1단계: @ComponentScan
  * 2단계: @EnableAutoConfiguration

### 1.1. @ComponentScan

* 현재 위치에서 하위 패키지의 Component들을 스캔한다.
* @Component
  * @Configuration @Repository @Service @Controller @RestController

### 1.2. @EnableAutoConfiguration

* Spring Boot의 자동 설정 메커니즘이다.
* Spring 메타 파일들(기본 설정들인 EnableAutoConfiguration)을 읽으며 Bean을 등록한다.
  * Web이나 Servlet 등 웹 프로젝트를 생성할 때 사용할 관련 기본 의존성 설정들.
* ``spring-boot-autoconfigure`` 의존성 내부의 spring.factories 파일에 정의된 클래스들이 자동으로 설정된다.
  * org.springframework.boot.autoconfigure.EnableAutoConfiguration 절 이하의 클래스 파일 Bean들.
* @ConditionalOnXxxYyyZzz
  * @Configuration 중 조건에 따라 등록한다.

### 1.3. 기타

> Application.java

```java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

* @SpringBootApplication에서 제공하는 일부 기능들을 다른 기능으로 대체하고 싶다면 커스텀하게 수정할 수 있다.

<br>

## 2. Custom Auto Configuration를 위한 XML

```xml
//라이브러리 패키지
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure-processor</artifactId>
    <optional>true</optional>
  </dependency>
</dependencies>

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.0.3.RELEASE</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

* xxx-Spring-Boot-Autoconfigure 모듈 : 자동 설정.
* xxx-Spring-Boot-Starter 모듈 : 필요한 의존성 정의.
  * 하나로 만들 때는 그냥 Starter로 정의한다.
* 위 예제 코드처럼 의존성을 주입한다.

<br>

## 3. 예제 코드

> HoloManConfiguration.java

```java
//라이브러리 패키지
@Configuration
public class HoloManConfiguration {

    @Bean
    public HoloMan holoMan() {
        return new HoloMan("holo", 5);
    }
}
```

> another-spring-boot-project.xml

```xml
//의존성 주입받는 프로젝트
<dependencies>
  <dependency>
    <groupId>me.glenn</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
</dependencies>
```

* HoloMan이라는 Bean을 리턴하는 @Configuration 설정 파일을 작성한다.
* src/main/resource/META-INF에 spring.factories 파일을 만든다.
  * ``org.springframework.boot.autoconfigure.EnableAutoConfiguration=\me.glenn.book.HoloManConfiguration``
* mvn install (로컬 패키징).
* 이후 다른 프로젝트에서 사용하려고 하는 프로젝트의 dependency를 추가한다.
  * 해당 Configuration을 가진 groupId, artifactId, version 등.
* 의존성을 주입받은 프로젝트에서는 HoloMan의 Bean을 등록하지 않아도, 해당 Bean을 사용할 수 있다.

### 3.1. 문제점

> AnotherProjectApplication.java

```java
//의존성을 주입받은 프로젝트
@Configuration
public class AnotherProjectApplication {

    @Bean
    public HoloMan holoMan() {
        return new HoloMan("holoAnother", 999);
    }
}
```

* A라는 프로젝트가 B 프로젝트의 AutoConfiguration을 사용할 때.
  * A와 B 둘다 HoloMan이라는 이름의 Bean이 존재한다면 B의 Bean을 이용한다.
  * ComponentScan을 먼저하고 난 다음 AutoConfiguration을 하기 때문에 B의 Bean이 A를 덮어쓴다.

### 3.2. Bean 덮어 씌우기 해결책

> HoloManConfiguration.java

```java
//라이브러리 패키지
@Configuration
public class HoloManConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public HoloMan holoMan() {
        return new HoloMan("holo", 5);
    }
}
```

* @ConditionalOnMissingBean 어노테이션을 통해, 해당 Bean이 없을 경우 AutoConfiguration을 통해 추가한다.
* C의 #ifndef와 비슷하다.

### 3.3. @ConfigurationProperties를 통한 해결

> application.properties

```properties
//의존성 주입받은 프로젝트
holoman.name=holoAnother
holoman.data=999
```

> HoloManProperites.java

```java
//라이브러리 패키지
@Getter @Setter
@ConfigurationProperties("holoman")
public class HoloManProperties {

    private String name;
    private int data;
}
```

> my-spring-boot-starter.xml

```xml
//라이브러리 패키지
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency>
```

> HoloManConfiguration.java

```java
//라이브러리 패키지
@Configuration
@EnableConfigurationProperties(HoloManProperties.class)
public class HoloManConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public HoloMan holoMan(HoloManProperties holoManProperties) {
        return new HoloMan(holoManProperties.getName(), holoManProperties.getData());
    }
}
```

* Bean을 재정의하는 수고스러움을 덜고 싶으면, @ConfigurationProperties를 사용한다.
* XML 파일은 자동 완성을 위한 추가 기능이다.
* 의존성을 주입받은 프로젝트는 Bean을 재정의할 필요 없이, 정의한 properties를 참조하여 Bean을 등록한다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
* [[SpringBoot] Spring 에서 자동설정의 이해와 구현 (AutoConfiguration)](https://ddingg.tistory.com/75)
