---
title: "유효성 검사 로직의 위치 및 예외 처리 전략"
excerpt: "도메인의 유효성 검사 로직은 어느 곳에 위치해야 할까?"
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-06-12
last_modified_at: 2021-06-12
---

## 1. 유효성 검사 : Persistence Layer vs Application Layer

지하철 노선도 관리 어플리케이션을 제작하는데 다음과 같은 테이블이 정의되어 있다.

> SQL

```sql
create table if not exists LINE
(
    id bigint auto_increment not null,
    name varchar(255) not null unique,
    color varchar(20) not null,
    primary key(id)
);
```

클라이언트가 지하철 노선을 등록할 때, 이름이 중복되는 경우 적절한 에러 메시지를 보여주려고 한다. 중복되는 노선 이름으로 Line 테이블에 데이터를 저장하려고 할 때 예외가 발생하면, @ExceptionHandler는 응답 코드 400과 적절한 에러 메시지를 클라이언트에게 반환하고 싶다.

유효성 검사 및 예외 처리 로직을 어느 곳에 위치해야 할까? 두 가지 시나리오가 존재한다.

### 1.1. Persistence Layer

먼저, DB 스키마 제약 조건으로 유효성 검사 및 예외 처리하는 방법을 고려할 수 있다.

name 칼럼은 DB 스키마 유일키 제약 조건이 걸려있다. 만약 DAO에서 중복되는 이름으로 데이터를 저장하려고 한다면, Spring JDBC는 SQLException을 감싼 RuntimeException인 DataAccessException을 반환할 것이다.

> LineDao.java

```java
public Long save(Line line) {
    SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(line);
    return simpleJdbcInsert.executeAndReturnKey(sqlParameterSource)
            .longValue();
}
```

> ControllerAdvice.java

```java
@ExceptionHandler(DataAccessException.class)
public ResponseEntity<String> handleSqlException(DataAccessException dataAccessException) {
    return ResponseEntity.badRequest()
            .body(dataAccessException.getMessage());
}
```

그러나 DataAccessException 혹은 SqlException 예외 자체를 바인딩해 응답을 작성하는 방법은 여러 한계점이 존재한다.

1. 예외 메시지를 세밀하게 조정할 수 없다.
  * 응답 본문에는 ``이미 존재하는 노선 이름입니다.``와 같은 커스텀한 예외 메시지가 아니라, DataAccessException의 예외 메시지가 담긴다.
    * 클라이언트에게는 별 도움이 되지 않는 영어 메시지일 확률이 높다.
2. 다른 테이블에서 발생하는 유일키 제약 위반과 구분할 수 없다.
  * Station 테이블에 중복된 이름의 데이터를 저장하려고 할 때는, ``이미 존재하는 역 이름입니다.``를 반환하고 싶어도 위 방식으로는 구분이 불가능하다.
3. 예외의 원인이 너무 다양하다.
  * DataAccessException이나 SqlException은 비단 DB 제약 위반과 같이 개발자가 예측 및 의도한 경우의 예외 뿐만 아니라, DB 연결 실패 등 다양한 원인에서 발생할 수 있다.
  * 공개할 필요가 없는 DB 에러 정보를 클라이언트에게 반환하는 것은 별 도움이 되지 않을 뿐더러, SQL Injection을 유도할 수 있는 보안 정보를 유출하기 쉽다.

> ControllerAdvice.java

```java
@ExceptionHandler(DuplicateKeyException.class)
public ResponseEntity<String> handleDuplicatedKeyException(DuplicateKeyException duplicateKeyException) {
    return ResponseEntity.badRequest()
            .body(duplicateKeyException.getMessage());
}

@ExceptionHandler(DataAccessException.class)
public ResponseEntity<Void> handleSqlException(DataAccessException dataAccessException) {
    LOGGER.error(dataAccessException.getMessage());
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .build();
}
```

JdbcTemplate의 메서드를 잘 살펴보면 유일키 제약 위반시 DuplicatedKeyException 혹은 DataIntegrityViolationException을 반환한다. 개발자가 발생을 예측한 구체적인 DB 에러는 적절히 처리해주고, 그 외의 DB 에러는 별도의 에러 메시지 없이 500 에러를 반환하고 로그를 찍도록 한다.

기존 방법보다는 낫지만 여전히 부족한 부분이 많다. 해당 유일키 제약 위반이 Line 테이블에서 발생했는지, Station 테이블에서 발생했는지 알 방법이 없다. 따라서 ``이미 존재하는 노선 이름입니다.`` 혹은 ``이미 존재하는 역 이름입니다.``와 같이 예외 메시지를 세밀하게 조정할 수 없다.

가장 큰 문제는 유효성 검사 및 예외 처리 관련 로직이 DB에 과하게 의존적이다. 먼저 ControllerAdvice에는 특정 예외에 종속적인 어드바이스가 많아지게 될 것이다. 예를 들어, 특정 ID에 해당하는 엔티티 조회 결과가 없는 경우를 생각해보자. JdbcTemplate의 ``queryForObject()`` 메서드는 조회 결과가 없을 때, EmptyResultDataAccessException을 반환할 것이다.

> ControllerAdvice.java

```java
@ExceptionHandler(EmptyResultDataAccessException.class)
public ResponseEntity<String> handleEntityNotFoundException(EmptyResultDataAccessException emptyResultDataAccessException) {
    return ResponseEntity.status(404)
            .body("해당 ID에 해당하는 엔티티가 존재하지 않습니다.");
}
```

그럴 때마다 위와 같이 특정 예외에 종속되는 어드바이스를 추가해야 한다. 프로젝트 규모가 커지면서 고려할 예외 상황이 많아진다면 ControllerAdvice 또한 비대해질 것이다. 덤으로 여전히 ``해당 노선이 존재하지 않습니다.`` 혹은 ``해당 역이 존재하지 않습니다.``와 같이 에러 메시지를 세밀하게 조정할 수 없다. 어디서 발생한 예외인지 구분할 수 없기 때문이다.

> DuplicatedNameException 계층 구조

```
DuplicatedNameException
|______________________ DuplicatedStationNameException
|______________________ DuplicatedLineNameException
```

> LineDao.java

```java
public Long save(Line line) {
    SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(line);
    try {
        return simpleJdbcInsert.executeAndReturnKey(sqlParameterSource)
                .longValue();    
    } catch (DuplicateKeyException e) {
           throw new DuplicatedLineNameException();
    }
}
```

> StationDao.java

```java
public Long save(Station station) {
    SqlParameterSource sqlParameterSource = new BeanPropertySqlParameterSource(station);
    try {
        return simpleJdbcInsert.executeAndReturnKey(sqlParameterSource)
                .longValue();    
    } catch (DuplicateKeyException e) {
           throw new DuplicatedStationNameException();
    }
}
```

> ControllerAdvice.java

```java
@ExceptionHandler(DuplicatedNameException.class)
public ResponseEntity<String> handleDuplicatedNameException(DuplicatedNameException duplicatedNameException) {
    return ResponseEntity.badRequest()
            .body(duplicatedNameException.getMessage());
}
```

예외가 상속을 통한 계층 구조를 형성하도록 한다. DAO에서 발생하는 예외를 try-catch를 통해 적절한 예외로 변환한다면, 다형성을 통해 하나의 어드바이저로 여러 예외를 잡을 뿐더러 예외 메시지를 세밀하게 조정할 수 있다. 그러나 유효성 검사 및 예외 처리 로직을 Persistence Layer에 놓는 바람에 try-catch를 반강제로 사용해야 한다. try-catch를 남용하면 코드 가독성도 나빠질 뿐더러, 성능 측면에서도 좋지 못하다.

특히 DB 스키마 제약 조건으로 유효성 검사 및 예외 처리하는 방법은 Java 코드만 보고는 프로젝트를 이해하기 어렵다는 문제가 있다. 코드를 이해하기 위해 DB 테이블 스키마까지 확인해야 하는 번거로움이 존재한다.

여러 측면에서 바라볼 때, 유효성 검사 및 예외 처리 로직을 DB 스키마 제약 조건에 전적으로 의존하여 진행하는 것은 무리가 있다.

### 1.2. Application Layer

리뷰어님들의 의견을 종합했을 때 다음과 같은 원칙을 세울 수 있었다.

* Application Layer에서 유효성 검사를 진행하는 것을 원칙으로 하고, 최후의 보루로 DB 제약 조건을 이용하자.
* DB 커넥션 등이 발생하긴 하겠지만 전체적으로 봤을 때 미미하다.
* 오히려 좀 더 빠른 시점에 에러를 발생시킬 수 있으며, Java 코드만 보고도 로직을 이해하기 쉽다.

> LineService.java

```java
public LineResponse saveLine(LineRequest request) {
    Line line = request.toEntity();
    lineDao.findByName(line.getName())
            .ifPresent(() -> {
                throw new DuplicatedLineNameException();
            });
    lineDao.save(line);
    //....
}

public LineResponse editLine(long id, LineRequest lineRequest) {
    Line line = lineDao.findById(id)
            .orElseThrow(LineNotFoundException::new);
    lineDao.findByColor(lineRequest.getColor())
            .orElseThrow(LineColorDuplicationException::new);
    //...
}
```

* ``findByXXX`` 메서드 조회 결과가 없을 때 발생하는 EmptyResultDataAccessException을 @ExceptionHandler가 핸들링하지 않고, DAO가 Optional 객체를 반환하도록 한다.
* 서비스 레이어가 중복일 때 혹은 중복이 아닐 때 등의 유효성 검사를 진행하도록 한다.
  * ``orElseThrow()`` 및 ``ifPresent()``와 같은 메서드 및 람다 표현식을 통해 Java 코드만 보고도 어떤 로직인지 쉽게 이해할 수 있다.

<br>

## 2. 예외 처리 전략

예외 클래스 또한 상속을 통해 계층 구조를 형성한다면, 다형성을 통해 적은 수의 @ExceptionHandler로 다양한 예외를 처리할 수 있을 뿐만 아니라 예외 코드 및 에러 메시지를 세밀하게 조정할 수 있다.

> exception 패키지 계층 구조

```
NotFound
  ㄴ NotFoundException
    ㄴ StationNotFoundException
    ㄴ LineNotFoundException
    ㄴ EmailNotFoundException

BadRequest
  ㄴ BadRequestException
    ㄴ DuplicationException
      ㄴ DuplicatedStationNameException
      ㄴ DuplicatedLineNameException
      ㄴ DuplicatedLineColorException
      ㄴ DuplicatedEmailException
    ㄴ InvalidValueException
      ㄴ InvalidAddableSectionException
      ...

Auth
  ㄴ AuthException
    ㄴ InvalidLoginInfoException
    ...
```

> ControllerAdvice.java

```java
@ExceptionHandler(NotFoundException.class)
public ResponseEntity<String> handleNotFoundException(NotFoundException notFoundException) {
    return ResponseEntity.notFound()
            .body(notFoundException.getMessage());
}

@ExceptionHandler(BadRequestException.class)
public ResponseEntity<String> handleBadRequestException(BadRequestException badRequestException) {
    return ResponseEntity.badRequest()
            .body(badRequestException.getMessage());
}


@ExceptionHandler(AuthException.class)
public ResponseEntity<String> handleBadRequestException(AuthException authException) {
    return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(authException.getMessage());
}
```

### 2.1. 생각해볼 문제

보기만 해도 커스텀 예외가 굉장히 많다. 프로젝트가 커질수록 관리하기 굉장히 힘들어질 것으로 보인다. 모든 상황마다 그 상황에서만 사용할 수 있는 예외를 생성하는 것이 적절할지 고민해보아야 한다.

### 2.2. 예외 패키지

현재 모든 예외들을 exception이라는 패키지에서 관리한다. 예외를 하나의 패키지에 모두 담아 관리하는 방식이 좋을까? 관련 아티클을 찾아 본 결과 그렇게 좋은 방식은 아닌 것 같다.

* 패키지는 한 개의 기능 단위를 의미하고, 커스텀 예외 또한 한 개의 기능 단위의 일부분이다.
* 따라서 예의를 발생시키는 곳(도메인 등) 같은 패키지에 속해 있어야 한다.
* 오히려 예외를 별도의 패키지로 그룹화하면 불필요한 내부 패키지 의존성을 유발한다.

즉, DuplicatedStationNameException은 exception 패키지가 아니라 ``station.domain``이나 ``station.service``에서 위치하는 것이 적절할 것으로 보인다.

### 2.3. Enum을 활용한 리팩토링

> BusinessException.java

```java
public class BusinessException extends RuntimeException {

    private final ExceptionStatus exceptionStatus;

    public BusinessException(ExceptionStatus exceptionStatus) {
        super(exceptionStatus.getMessage());
        this.exceptionStatus = exceptionStatus;
    }

    public int getStatus() {
        return exceptionStatus.getStatus();
    }
}
```

> ExceptionStatus.java

```java
public interface ExceptionStatus {

    int getStatus();

    String getMessage();
}
```

> AuthorizationException.java

```java
public class AuthorizationException extends BusinessException {

    public AuthorizationException(ExceptionStatus exceptionStatus) {
        super(exceptionStatus);
    }
}
```

> AuthorizationExceptionStatus.java

```java
public enum AuthorizationExceptionStatus implements ExceptionStatus {
    WRONG_PASSWORD(401, "비밀번호가 틀렸습니다."),
    WRONG_EMAIL(401, "잘못된 이메일입니다."),
    LOGIN_REQUIRED(401, "로그인이 필요합니다."),
    INVALID_TOKEN(401, "유효하지 않은 토큰입니다.");

    private final int status;
    private final String message;

    AuthorizationExceptionStatus(int status, String message) {
        this.status = status;
        this.message = message;
    }

    @Override
    public int getStatus() {
        return status;
    }

    @Override
    public String getMessage() {
        return message;
    }
}
```

> AuthService.java

```java
public void validate(String credentials) {
    boolean isValidToken = jwtTokenProvider.validateToken(credentials);
    if (!isValidToken) {
        throw new AuthorizationException(AuthorizationExceptionStatus.INVALID_TOKEN);
    }
}
```

> ControllerAdvice.java

```java
@ExceptionHandler(BusinessException.class)
public ResponseEntity<String> handleBusinessException(BusinessException businessException) {
    LOGGER.info(businessException.getMessage());
    return ResponseEntity.status(businessException.getStatus())
            .body(businessException.getMessage());
}
```

비슷한 예외 상황이더라도 상태 코드를 다르게 반환해야 하는 경우가 존재할 수 있다.

* 예외 계층을 활용한 기존의 방식은 BadRequestException 휘하의 모든 예외들이 일괄적으로 400 상태 코드를 반환한다.
* 만약 요구 사항 변경으로 인해 일부 예외들은 다른 상태 코드를 보여줘야 한다면 계층 수정부터 많은 수정이 발생하게 된다.

Enum을 활용하면 이러한 문제를 다소 해결할 수 있지 않을까 생각해본다. 다만 서비스 로직에서 예외 호출시 주입하는 예외 상수를 직접적으로 의존하고 있는 점이 걸린다. 😭

<br>

---

## References

* [Spring Guide - Exception 전략](https://cheese10yun.github.io/spring-guide-exception/)
* [@ControllerAdvice, @ExceptionHandler를 이용한 예외처리 분리, 통합하기(Spring에서 예외 관리하는 방법, 실무에서는 어떻게?)](https://jeong-pro.tistory.com/195)
* [Should Exceptions be placed in a separate package?](https://stackoverflow.com/questions/825281/should-exceptions-be-placed-in-a-separate-package)
