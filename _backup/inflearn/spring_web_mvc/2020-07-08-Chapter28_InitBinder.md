---
title: "[Spring Web MVC] 28장 : @InitBinder"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:30:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. @InitBinder

> SampleController.java

```java
@InitBinder
public void initBinder(WebDataBinder webDataBinder) {
    webDataBinder.setDisallowedFields("id");
    webDataBinder.addCustomFormatter(new EventFormatter());
    webDataBinder.addValidators(new EventValidator());
}
```

> EventValidator.java

```java
public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> aClass) {
        return Event.class.isAssignableFrom(aClass);
    }

    @Override
    public void validate(Object o, Errors errors) {
        Event event = (Event)o;
        if (event.getName().equalsIgnoreCase("AAA")){
            errors.rejectValue("name", "wrongValue", "cannot AAA");
        }
    }
}
```

> SampleController.java

```java
//InitBinder를 사용하지 않고 Bean으로 Validator를 등록
@Autowired
EventValidator eventValidator;

@GetMapping("/test")
public void test(Event event, BindingResult bindingResult) {
    //로직...
    eventValidator.validate(event, bindingResult);
    //원하는 시점에서 검증
}
```

* 특정 컨트롤러에서 바인딩 또는 검증 설정을 커스텀하게 변경하고 싶을 때 사용한다.
* 특정 모델 객체에만 바인딩 또는 Validator 설정을 적용하고 싶은 경우 `@InitBinder(“event”)` 와 같이 명시한다.
* 바인딩과 포매터 및 검증 관련 설정을 할 수 있으며, Validator를 별도의 Bean으로 등록하여 원하는 시점에 핸들러 내부에서 실행하는 대안도 있다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
