---
title:  "[Spring Boot 개념과 활용] 3장 : 내장 웹 서버"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:05:00-05:00
---

## 1. 내장 서블릿 컨테이너

* Spring Boot는 서버가 아니다.
  * 톰캣 객체 생성.
  * 포트 설정.
  * 톰캣에 컨텍스트 추가.
  * 서블릿 만들기.
  * 톰캣에 서블릿 추가.
  * 컨텍스트에 서블릿 맵핑.
  * 톰캣 실행 및 대기.
* 이 모든 과정을 보다 상세히 또 유연하고 설정하고 실행해주는게 Spring Boot의 자동 설정이다.
  * ServletWebServerFactoryAutoConfiguration(서블릿 웹 서버 생성).
  * TomcatServletWebServerFactoryCustomizer(서버 커스터마이징).
  * DispatcherServletAutoConfiguration.
    * 서블릿을 만들고 등록한다.
    * 서블릿 컨테이너는 Tomcat, Jetty 등 변경될 수 있기 때문에 DispatcherSevrlet 생성과 분리되어 있다.

<br>

## 2. 다른 서블릿 컨테이너 사용하기

> pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Exclude the Tomcat dependency -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- Use Jetty instead -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

* 레퍼런스를 참조하면 상세하게 설명되어 있다.
* 다른 서블릿으로 제티나 언더토를 사용할 수 있다.

<br>

## 3. 웹 서버 사용하지 않기

> application.properties

```properties
spring.main.web-application-type=none
```

<br>

## 4. Port 변경

> application.properties

```properties
server.port=7070
//0이면 Random Port
```

<br>

## 5. 런타임 시 HTTP Port 콜백

> PortListener.java

```java
@Component
public class PortListener {

    @EventListener
    public void onApplicationEvent(ServletWebServerInitializedEvent event) {
        ServletWebServerApplicationContext applicationContext = event.getApplicationContext();
        System.out.println(applicationContext.getWebServer().getPort());
    }
}
```

* EventListener로 웹 서버가 런타임될 때 HTTP 관련 콜백을 진행하여 Port 값을 확인한다.

<br>

## 6. HTTPS 사용하기

> application.properties

```properties
server.ssl.key-store=keystore.p12
server.ssl.key-store-password=123456
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=tomcat
//만약 추가 포트 설정을 한다면
server.port=8443
```

> generate-keystore.sh

```sh
keytool -genkey
  -alias tomcat
  -storetype PKCS12
  -keyalg RSA
  -keysize 2048
  -keystore keystore.p12
  -validity 4000
```

> ShellScript

```sh
curl -I --http2 http://localhost:8080 //close
curl -I --http2 http://localhost:8080/hello //close
curl -I --http2 https://localhost:8080/hello //인증서 에러
curl -I -K --http2 https://localhost:8080/hello //정상 확인
//아직 http2 설정을 하지 않았다면 http1로 확인
```

> Application.java

```java
@Bean
public ServletWebServerFactory servletContainer() {
  TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
  tomcat.addAdditionalTomcatConnectors(createStandardConnector());
  return tomcat;
}

private Connector createStandardConnector() {
  Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
  connector.setPort(8443);
  return connector;
}
```

* HTTPS 요청이 아닌 요청들은 거부하게 된다.
* HTTPS를 사용하면 HTTP를 받을 수 있는 커넥터가 없어 사용이 불가능하다.
  * 레퍼런스를 참고하여 2가지 요청을 모두 지원할 수 있도록 커넥터를 추가할 수 있다.
  * 위 예제는 8443 포트를 통해 https를 사용하며, 8080은 일반 http를 지원한다.
  * 이 경우 application.properties에 port를 추가해야 한다.

<br>

## 7. HTTP2 undertow

> application.properties

```properties
server.http2.enabled=true
```

* XML에서 톰캣을 제외시키고 undertow를 추가한다.
* 이후 리눅스 상에서 curl을 통해 HTTP 통신을 확인하면 HTTP2로 잡힌다.
* 톰캣으로 HTTP2 프로토콜을 사용하기 위해서는 jdk & 톰캣의 버전과 IDE 추가 설정이 필요하다.
  * 동영상 강의 및 레퍼런스를 참조하자.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
