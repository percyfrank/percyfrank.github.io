---
title: "[Spring Web MVC] 25장 : @RequestBody & HttpEntity"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:24:00-05:00
---

## 1. @RequestBody

> EventApi.java

```java
@RestController
@RequestMapping("/api/events")
public class EventApi {

    @PostMapping
    public Event createEvent(@RequestBody Event event) {
        //save Event
        return event;
    }

    @PostMapping
    public Event createEvent(HttpEntity<Event> request) {
        MediaType contentType = request.getHeaders().getContentType();
        return request.getBody();
    }
}
```

> SampleControllerTest.java

```java
@Autowired
ObjectMapper objectMapper;

@Test
public void createEvent() throws Exception {
    Event event = new Event();
    event.setName("jipark");
    event.setId(10);
    String jsonMessage = objectMapper.writeValueAsString(event);

    this.mockMvc.perform(post("/api/events")
        .contentType(MediaType.APPLICATION_JSON_UTF8)
        .content(jsonMessage))
        .andDo(print())
        .andExpect(status().isOk())
        .andExpect(jsonPath("name").value("jipark"))
        .andExpect(jsonPath("id").value(10));
}
```

* 요청 본문(body)에 들어있는 메시지를 HandlerAdapter가 HttpMessageConveter를 통해 객체로 변환해 받아올 수 있다.
* @ModelAttribute와 동일하게 검증 및 바인딩 에러를 다룰 수 있다.
  *	@Valid 또는 @Validated를 사용해서 값을 검증 할 수 있다.
  *	BindingResult 아규먼트를 사용해 코드로 바인딩 또는 검증 에러를 확인할 수 있다.
*	HttpEntity는 @RequestBody와 비슷하지만 추가적으로 요청 헤더 정보를 사용할 수 있다.
  * 적합한 Body 타입을 지정해야 한다.
*	ObjectMapper를 통해 객체를 JSON 문자열로, JSON 문자열을 객체로 만들 수 있다.

<br>

## 2. HttpMessageConverter

* Spring MVC 설정(WebMvcConfigurer)에서 설정할 수 있다.
*	configureMessageConverters : 기본 메시지 컨버터를 사용하지 않고 새로운 컨버터만 사용한다.
*	extendMessageConverters : 기본 메시지 컨버터를 사용하면서 새로운 컨버터를 추가한다.
*	기본 컨버터 : WebMvcConfigurationSupport.addDefaultHttpMessageConverters.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
