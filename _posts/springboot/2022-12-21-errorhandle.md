---
title:  "SpringBoot 예외 처리의 시작" 
excerpt: "에외처리를 해보자"

categories:
  - Springboot
tags:
  - Springboot

date: 2022-12-21
last_modified_at: 2022-12-21

---

서버에서 발생하는 오류는 기본적으로 `HTTP 500 Interval Server Error`를 반환한다.

하지만, 개발자가 예외처리를 통해 클라이언트에게 개발자가 정해둔 에러 메세지라던가 상황별로 분류해놓은 상태코드를 반환할 수 있다.

이를 위한 기능이 Spring에선 `@ExceptionHandler` 라는 어노테이션으로 가능하다.

`@ExceptionHandler`는 AOP를 이용한 예외 처리 방식으로 해당 어노테이션을 메서드 위에 선언하고 특정 예외 클래스를 지정하면, 해당 예외가 발생했을 때 메서드에 정의한 로직으로 처리할 수 있다.

예를 들어, 다음은 NullPointerException을 처리하는 Handler이다. 이렇게, 어떤 예외를 처리할 것인지 명시해주면 해당 예외를 처리할 수 있다.
```java
@ExceptionHandler(NullPointerException.class)
public ResponseEntity<?> NullPointerExceptionHandler(Exception e) {
  ...
}
```

이 Handler 어노테이션을 이용해서 일반 Controller 클래스에 존재하는 메서드에 선언할 경우, 해당 Controller에만 적용이 된다. 물론, 해당 컨트롤러와 의존관계를 맺고 있는 곳에서 발생한 예외도 처리한다.

**하지만, 아예 관계없는 다른 컨트롤러에서 발생하는 예외는 처리할 수 없다!!**

**이건 되게 심각한 문제다...**

>컨트롤러가 한 두개가 아닐텐데, 의존관계를 맺고 있지 않은 컨트롤러마다 Handler 어노테이션을 적용시켜야 된다는 소리다.

다행히도 스프링은 예외 처리 로직을 분리해서 하나의 모듈처럼 사용할 수 있다. 

`@ControllerAdvice`와 `@RestControllerAdvice` 어노테이션이 바로 그 역할을 할 수 있다.

전역 처리를 위한 클래스를 만들어두고 어노테이션을 이용해 모든 클래스에 전역으로 적용이 가능하다. 



다음과 같이 모든 컨트롤러에 예외처리 핸들러를 적용할 수 있다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

}
```

추가적으로, 특정 컨트롤러에만 적용할 수도 있다.
```java
@RestControllerAdvice(annotations = xxController.class)
public class GlobalExceptionHandler {

}
```
>`@ControllerAdvice`와 `@RestControllerAdvice`의 차이는 단순히 예외 처리를 할 것인가 아니면 응답을 JSON으로 리턴까지 해줄 것인가에 따라 나누어진다.<br>
후자의 경우가 `@RestControllerAdvice`어노테이션의 기능이고, 해당 어노테이션엔 `@ResponseBody`가 합쳐져 있는 어노테이션이라 가능하다.

---

그럼 이제 에러 처리에 관한 대부분의 내용을 살펴봤으니 직접 에러 처리를 해보자.

순서는 다음과 같다.
1. enum 클래스의 에러 코드
2. 예외 발생 시 응답하는 에러 정보 반환 클래스
3. 에러 정보 반환 클래스를 한번 감싸줄 응답 클래스
4. 사용자 정의 예외 클래스
5. 전역 처리할 handler
6. 에러처리 확인 

<br>

### 1.enum 클래스의 에러 코드
---
`ErrorCode.java`
```java
@AllArgsConstructor
@Getter
public enum ErrorCode {

    DUPLICATED_USER_NAME(HttpStatus.CONFLICT, "UserName이 중복됩니다."),
    USERNAME_NOT_FOUND(HttpStatus.NOT_FOUND,"Not founded"),
    INVALID_PASSWORD(HttpStatus.UNAUTHORIZED, "패스워드가 잘못되었습니다."),
    INVALID_TOKEN(HttpStatus.UNAUTHORIZED, "잘못된 토큰입니다."),
    TOKEN_NOT_FOUND(HttpStatus.UNAUTHORIZED, "토큰이 존재하지 않습니다."),
    INVALID_PERMISSION(HttpStatus.UNAUTHORIZED, "사용자가 권한이 없습니다."),
    POST_NOT_FOUND(HttpStatus.NOT_FOUND, "해당 포스트가 없습니다."),
    DATABASE_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "DB에러");

    private HttpStatus status;
    private String message;
}
```
제일 먼저 클라이언트에게 보내줄 에러 코드를 정의해야 한다. 

에러 이름과 HTTP 상태 및 메세지를 가지고 있는 에러 코드 클래스를 만들어 보도록 하자. 나의 프로젝트는 간단한 프로젝트라 추상화를 하지 않고 바로 에러코드 enum 클래스를 만들었다.

하지만 경우에 따라, 다음과 같이 애플리케이션에서 전역적으로 사용되는 CommonErrorCode와 특정 도메인에 대해 구체적으로 내려가는 UserErrorCode로 나누고, 인터페이스를 이용해 추상화할 수도 있다.

```java
public interface ErrorCode {

    String name();
    HttpStatus getHttpStatus();
    String getMessage();

}

@Getter
@RequiredArgsConstructor
public enum CommonErrorCode implements ErrorCode {

    INVALID_PARAMETER(HttpStatus.BAD_REQUEST, "Invalid parameter included"),
    RESOURCE_NOT_FOUND(HttpStatus.NOT_FOUND, "Resource not exists"),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "Internal server error"),
    ;

    private final HttpStatus httpStatus;
    private final String message;
}

@Getter
@RequiredArgsConstructor
public enum UserErrorCode implements ErrorCode {

    INACTIVE_USER(HttpStatus.FORBIDDEN, "User is inactive"),
    ;

    private final HttpStatus httpStatus;
    private final String message;
}

```

### 2. 예외 발생 시 응답하는 에러 정보 반환 클래스
---
에러 정보를 반환할 때에도 일관된 형식으로 반환하기 위한 클래스이다.

`ErrorResponse`
```java
@Getter
public class ErrorResponse {

    private ErrorCode errorCode;
    private String message;

    public ErrorResponse(ErrorCode errorCode) {
        this.errorCode = errorCode;
        this.message = errorCode.getMessage();
    }
}
```

### 3. 에러 정보 반환 클래스를 한번 감싸줄 응답 클래스
---

에러 정보 반환 클래스를 한 번 더 감싸줄 응답 클래스를 만든 이유는, 에러를 반환하는 것 뿐 아니라 성공 상황에서도 일관성있게 반환하도록 하기 위해 더 상위의 클래스를 만들었다.

`Response`
```java
@AllArgsConstructor
@Getter
public class Response<T> {
    private String resultCode;
    private T result;

    public static <T> Response<T> success(T result) {
        return new Response("SUCCESS", result);
    }

    public static <T> Response<T> error(String resultCode, T result) {
        return new Response(resultCode, result);
    }
}
```

### 4. 사용자 정의 예외 클래스
---
우리가 발생한 예외를 처리해줄 예외 클래스를 추가한다. 

Unchecked 예외(런타임 예외)를 상속받는 예외 클래스를 다음과 같이 추가했다. 

Unchecked 예외를 상속받은 이유는 Checked 예외는 예외 처리를 강제한다. 그렇기 때문에 만약 Checked 예외로 한다면 불필요하게 throws가 전파될 것이다.

그렇기 때문에 내가 정의한 예외클래스는 Unchecked 예외를 상속하게 하고, Checked 예외는 전역 처리 Handler에서 따로 처리한다.

`AppException`
```java
@AllArgsConstructor
@Getter
public class AppException extends RuntimeException {

    private ErrorCode errorCode;
}
```

### 5. 전역 처리할 handler
---
Spring엔 예외를 미리 처리해둔 `ResponseEntityExceptionHandler`를 추상 클래스로 제공한다.

이 클래스에 예외에 대한 ExceptionHandler가 모두 구현되어 있기 때문에 상속받아 에러 정보 반환을 위해 메서드만 오버라이딩하면 된다.

하지만, 나는 그렇게 하지 않고, 직접 `GlobalExceptionHandler`를 만들었다.

다음과 같이 Unchecked 예외를 상속한 내가 정의한 `AppException`클래스 말고, Checked 예외인 `SQLException`클래스도 처리하도록 하였다.

`GlobalExceptionHandler`
```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(AppException.class)
    public ResponseEntity<?> appExceptionHandler(AppException e) {
        log.error("AppException : {}",e.getErrorCode());
        ErrorResponse errorResponse = new ErrorResponse(e.getErrorCode());
        return ResponseEntity.status(e.getErrorCode().getStatus())
                .body(Response.error("ERROR",errorResponse));
    }

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<?> runtimeExceptionHandler(RuntimeException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(e.getMessage());
    }

    @ExceptionHandler(SQLException.class)
    public ResponseEntity<?> sqlExceptionHandler(SQLException e){
        ErrorResponse errorResponse = new ErrorResponse(ErrorCode.DATABASE_ERROR);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(Response.error("ERROR", errorResponse));
    }
}
```

### 6. 에러처리 확인
---
`PostApiController.java`
```java
@GetMapping("/{postsId}")
public Response<PostDetailResponse> postDetail(@PathVariable("postsId") Integer id) {
    PostDetailResponse post = postService.findPost(id);
    log.info("포스트 상세 조회 성공");
    return Response.success(post);
}
```

`PostService.java`
```java
  @Transactional(readOnly = true)
  public PostDetailResponse findPost(Integer id) {
      // post 조회
      Post post = postRepository.findById(id)
              .orElseThrow(() -> new AppException(ErrorCode.POST_NOT_FOUND));

      return PostDetailResponse.of(post);
  }

```
특정 게시글을 조회하는 기능을 진행해보겠다. 

Service 코드에서 조회하려는 게시글이 없다면 내가 정의한 ErrorCode에서 `POST_NOT_FOUND`를 반환하도록 처리했다. 제대로 반환되는지 확인해보자.

![image](https://user-images.githubusercontent.com/85394884/210174548-60c7849f-b812-4efb-8934-248f344f4654.png)

내가 의도한 에러 반환이다.

<br>

---

## Refernces

* [Springboot Exception Handling](https://samtao.tistory.com/42)
* [예외처리](https://velog.io/@hellonayeon/spring-boot-global-exception)
* [스프링의 다양한 예외 처리 방법(ExceptionHandler, ControllerAdvice 등) 완벽하게 이해하기 - (1/2)](https://mangkyu.tistory.com/204)
* [@RestControllerAdvice를 이용한 Spring 예외 처리 방법 - (2/2)](https://mangkyu.tistory.com/205)
* [체크 예외(Check Exception)와 언체크 예외/런타임 예외 (Uncheck Exception, Runtime Exception)의 차이와 올바른 예외 처리 방법](https://mangkyu.tistory.com/152)
* [Spring Guide - Exception 전략](https://cheese10yun.github.io/spring-guide-exception/)