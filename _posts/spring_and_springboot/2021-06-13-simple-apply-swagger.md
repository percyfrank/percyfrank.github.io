---
title: "Spring Boot Swagger 적용 및 JWT 토큰 사용 설정"
excerpt: "Swagger로 API 문서화 및 테스트를 진행해보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-13
last_modified_at: 2021-06-13
---

## 1. 설정

> build.gradle

```groovy
implementation "io.springfox:springfox-boot-starter:3.0.0"
```

> SwaggerConfig.java

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .host("http://localhost:8080")
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
```

* 어플리케이션을 배포하면 ``host()`` 또한 배포 도메인으로 변경되어야 한다.

> WebConfig.java

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(loginInterceptor())
            .addPathPatterns("/**")
            .excludePathPatterns("/members", "/login/token", "/paths",
                    "/v2/api-docs",
                    "/swagger-resources/**",
                    "/swagger-ui/**",
                    "/webjars/**");
}
```

* 로컬에서 편하게 사용하기 위해 Swagger 관련 정적 자원들에 대해서는 로그인 인터셉터를 거치지 않도록 경로를 제외시킨다.
* Swagger 2.0은 기본 페이지가 ``domain/swagger-ui.html``이었으나, 3.0부터는 ``domian/swagger-ui/`` 혹은 ``domain/swagger-ui/index.html``로 변경되었다.
* 설정을 마치면 ``http://localhost:8080/swagger-ui/`` 접속시 Swagger 페이지가 정상 로드된다.

<br>

## 2. JWT 사용 설정

Swagger로 Postman처럼 API를 테스트할 수 있다. 그런데 어플리케이션 로직이 JWT 토큰과 같은 인증 정보를 필요로 하는 경우가 있다. 간단한 설정을 통해 Swagger에서도 JWT 사용을 설정해본다.

> SwaggerConfig.java

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .host("http://localhost:8080")
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build()
                .securityContexts(Arrays.asList(securityContext()))
                .securitySchemes(Arrays.asList(apiKey()));
    }

    private SecurityContext securityContext() {
        return SecurityContext.builder()
                .securityReferences(defaultAuth())
                .forPaths(PathSelectors.any())
                .build();
    }

    private List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        return Arrays.asList(new SecurityReference("JWT", authorizationScopes));
    }

    private ApiKey apiKey() {
        return new ApiKey("JWT", "Authorization", "header");
    }
}
```

![image](https://user-images.githubusercontent.com/56240505/121799020-a6abb980-cc64-11eb-8cdf-b9f12e04a744.png)

* 설정한 다음 Swagger 페이지로 이동하면 Authorize 버튼이 활성화되어 있다.
* Access Token을 발급받은 뒤, Value에 ``Bearer ${credential}``를 입력하면 API 테스트하는 동안 해당 토큰이 요청 헤더에 자동으로 삽입된다.
* SecurityScheme을 더 세부적으로 설정하면 Oauth2, Bearer 등을 지정할 수 있을 것으로 보이는데 나중에 API 문서를 정리해야겠다.

<br>

---

## References

* [Setting Up Swagger 2 with a Spring REST API](https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)
* [Springfox Reference Documentation](https://springfox.github.io/springfox/docs/current/#migrating-from-existing-2-x-version)
* [[SpringBoot] Swagger를 통한 REST 요청에 전역 jwt 인증 설정 하기](https://velog.io/@livenow/SpringBoot-Swagger%EB%A5%BC-%ED%86%B5%ED%95%9C-REST-%EC%9A%94%EC%B2%AD%EC%97%90-%EC%A0%84%EC%97%AD-jwt-%EC%9D%B8%EC%A6%9D-%EC%84%A4%EC%A0%95-%ED%95%98%EA%B8%B0)
