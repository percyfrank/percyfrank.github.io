---
title:  "[Spring Framework 핵심 기술] 8장 : MessageSource"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:13:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. MessageSource

> messages.properties

```properties
greeting=hello {0}
```

> messages_ko_KR.properties

```properties
greeting=안녕 {0}
```

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MessageSource messageSource;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(messageSource.getMessage("greeting", new String[]{"jipark"}, Locale.KOREA)); //안녕 jipark
        System.out.println(messageSource.getMessage("greeting", new String[]{"jipark"}, Locale.getDefault())); //hello jipark
    }
}
```

*	국제화(i18n) 기능을 제공하는 인터페이스이다.
*	MessageSource를 구현하는 ApplicationContext의 `getMessage(String code, Object[] args, String, default, Locale, loc)` 메소드 등을 이용한다.
*	Spring Boot를 이용하면 자동으로 Bean이 등록되어 별다른 설정 없이 message 관련 properties를 이용할 수 있다.

<br>

## 2. 릴로딩 기능이 있는 MessageSource

> Application.java

```java
@Bean
public MessageSource messageSource() {
      var messageSource = new ReloadableResourceBundleMessageSource();
      messageSource.setBasename("classpath:/messages");
      messageSource.setDefaultEncoding("UTF-8");
      messageSource.setCacheSeconds(3);
      return messageSource;
}
```

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
