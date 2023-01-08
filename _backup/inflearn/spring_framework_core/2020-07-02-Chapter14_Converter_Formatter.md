---
title:  "[Spring Framework 핵심 기술] 14장 : Converter와 Formatter"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:19:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Converter

> EventConverter.java

```java
public class EventConverter {

  @Component
  public class StringToEventConverter implements Converter<String, Event> {
      @Override
      public Event convert(String source) {
          Event event = new Event();
          event.setId(Integer.parseInt(source));
          return event;
      }
  }

  @Component
  public class EventToStringConverter implements Converter<Event, String> {
      @Override
      public String convert(Event source) {
          return event.getId().toString();
      }
  }
}
```

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebConfigurer {

  @Override
  public void addFormatters(FormatterRegistry registry) {
      registry.addConverter(new EventConverter.StringToEventConverter());
  }
}
```

* S 타입을 T 타입으로 변환할 수 있는 매우 일반적인 변환기이다.
* 상태 정보가 없어 Thread-Safe 하기 때문에 Bean 등록도 할 수 있다.
* Spring Boot 없이 사용하는 경우 Web 관련 Config 파일의 ConverterRegistry에 등록해서 사용한다.

<br>

## 2. ConversionService

![ConversionService](https://user-images.githubusercontent.com/56240505/80094846-ee274000-85a1-11ea-9d75-849e43a08e59.png)

* ConversionService는 런타임 시 타입 변환 로직을 실행하는 통홥된 API를 제공한다.
  * Spring MVC, DataBinder를 통한 Bean(value) 설정, SpEL 등에서 이를 사용한다.
* 무상태 특징으로 인해 해당 인터페이스를 통해서 Thread-Safe하게 타입을 변환할 수 있다.
* DefaultFormattingConversionService.
	* FormatterRegistry.
	* ConversionService.
	* 여러 기본 Converter 및 Formatter를 등록 해준다.
* Spring Boot는 DefaultFormattingConversionSerivce를 상속하여 만든 WebConversionService를 Bean으로 등록해준다.
  * Spring MVC에서는 WebConfig에 Formatter 및 Converter를 등록해야한다.
    * ConversionService가 등록되어 있지 않다면 기본적으로 PropertyEditor를 사용한다.
  * Spring Boot는 Formatter 및 Converter Bean을 찾아 자동으로 등록해 준다.
  * 자주 쓰이는 자료형간의 변환에 대해 Spring이 기본적으로 제공하는 Formatter를 사용한다.

<br>

## 3. Formatter

> EventFormatter.java

```java
@Component
public class EventFormatter implements Formatter<Event> {

    @Autowired
    MessageSource messageSource;

    @Override
    public Event parse(String text, Locale locale) throws ParseException {
        Event event = new Event();
        int id = Integer.parseInt(text);
        event.setId(id);
        return event;
    }

    @Override
    public String print(Event object, Locale locale) {
        return object.ge().toString();
    }
}
```

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatters(new EventFormatter());
    }
}
```

* PropertyEditor 대체제이며, Object와 String 간의 변환을 담당한다.
* 문자열을 Locale에 따라 다국화하는 기능도 제공한다.
  * Formatter는 Converter가 제공하지 않는 추가 유용한 기능을 다수 제공한다.
* 상태 정보 없어 Thread-Safe하기 때문에 Bean 등록도 할 수 있다.
* 다른 Bean을 주입받아 MessageSource를 응용할 수 있다.
* Spring의 경우 Web 관련 Config 파일의 FormatterRegistry에 등록해서 사용한다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
