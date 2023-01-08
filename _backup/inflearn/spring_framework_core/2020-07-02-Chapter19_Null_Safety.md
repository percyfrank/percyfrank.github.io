---
title:  "[Spring Framework 핵심 기술] 19장 : Null-Safety"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:24:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Null-Safety

> Test.java

```java
public class Test {

  	private String id;

  	@NonNull
  	public String helloId(@NonNull Stirng id) {
        return "hello " + id;
  	}
}
```

* IDE의 지원을 받아 컴파일 시점에 최대한 NullPointerException을 방지하는 것이 목적이다.
* IDE에 부수적인 설정을 해야 IDE 상에서 Warning을 인식할 수 있다.
	* IDE에서 Preferences - Configure Annotations의 Nullable/NonNull 설정에 Spring의 기능을 추가해준다.
* 패키지 레벨에서 @NonNull을 설정하고 Null이 발생할 수 있는 부분만 @Nullable을 설정하는 등의 코딩이 가능하다.
* Parameter 뿐만 아니라 메소드에도 Annotation을 붙이면 Return Type에도 Null-Safety가 적용된다.
* 관련 Annotation.
	* @NonNull
	* @Nullable
	* @NonNullApi(패키지 레벨 설정)
	* @NonNullFields(패키지 레벨 설정)

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
