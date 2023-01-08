---
title: "Login Interceptor 사용시 발생하는 CORS 이슈"
excerpt: "Filter 및 Interceptor 등의 동작 순서에 대해 유념하자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-10
last_modified_at: 2021-06-10
---

## 1. Login Interceptor의 도입과 CORS 에러

> LoginInterceptor.java

```java
public class LoginInterceptor implements HandlerInterceptor {

    private final AuthService authService;

    public LoginInterceptor(AuthService authService) {
        this.authService = authService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String credentials = AuthorizationExtractor.extract(request);
        authService.validate(credentials);
        return true;
    }
}
```

* 프론트 서버와 백 서버를 분리하고, 로그인 검증이 필요한 API들은 모두 Login Interceptor가 일괄적으로 JWT 토큰을 체크하도록 구현했다.
* Interceptor를 추가하니 기존에 잘 동작하던 기능들이 CORS 에러가 발생하기 시작했다.

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("*")
                .allowedOriginPatterns("*");
    }
}
```

* 기존에는 잘 동작했기에, WebMvcConfigurer 및 @CrossOrigin 등 CORS 맵핑 설정이 원인은 아닌듯 보였다.

<br>

## 2. 해결책

예전에 [CORS](https://xlffm3.github.io/network/cors/)를 학습할 때 꼼꼼하게 정리하지 않은 나의 불찰이다. 브라우저가 Options 메서드인 Preflight 요청을 보내고 나서, 두 가지 조건이 만족해야 Actual Request를 보낸다.

* Preflight Response의 응답 상태 코드가 200이어야 할 것.
* Preflight Response의 Access-Control-Allow-Origin 헤더에 Request의 출처가 명시되어야 할 것.

Spring CORS 설정은 Filter로 동작한다. 플로우는 대충 ``Http Request -> Filter -> Interceptor -> Controller``가 될 것이다. Options 메서드 요청은 해당 URI를 처리할 수 있는 메서드 정보 등을 확인하는 용도일뿐, 실제 서버 사이드 API를 실행시키지 않는다. 문제는 Interceptor 단이다.

> LoginInterceptor.java

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    String credentials = AuthorizationExtractor.extract(request);
    authService.validate(credentials);
    return true;
}
```

* Options Preflight 요청에는 JWT 액세스 토큰을 담은 Authorization 헤더가 존재하지 않는다.
  * ``Authorization : Bearer {credentials}``
  * 따라서 credentials는 null이고, ``validate()`` 메서드에서 예외가 발생한다.
* 이와 같은 유효성 검사 로직 때문에 401 혹은 500 에러가 발생하고, 브라우저는 이를 CORS 위반으로 인식해서 에러가 발생한 것이다.

> LoginInterceptor.java

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    if (HttpMethod.OPTIONS.matches(request.getMethod())) {
        return true;
    }
    String credentials = AuthorizationExtractor.extract(request);
    authService.validate(credentials);
    return true;
}
```

* Options Preflight 요청일 때는 유효성 검사 로직을 타지 않도록 변경해주니 문제가 해결되었다.

<br>

---

## References
