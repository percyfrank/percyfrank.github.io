---
title: "[Spring Web MVC] 23장 : FlashAttributes"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:22:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. FlashAttributes

> SampleController.java

```java
@PostMapping("/events/form/id")
public String eventId(@Validated @ModelAttribute Event event,
                      BindingResult bindingResult,
                      SessionStatus sessionStatus,
                      RedirectAttributes redirectAttributes) {
    if (bindingResult.hasErrors()) {
        return "form-id";
    }
    sessionStatus.setComplete();
    redirectAttributes.addFlashAttribute("newEvent", event);
    return "redirect:/events/list";
}

@GetMapping("/events/list")
public String getList(Model model,
                      @ModelAttribute("newEvent") Event newEvent,
                      @SessionAttribute LocalDateTime visitTime) {
    System.out.println(visitTime);
    Event event = new Event();
    List<Event> eventList = new ArrayList<>();
    event.setName("spring");
    eventList.add(event);
    eventList.add(newEvent);
    //or Event newEvent = (Event) model.asMap().get("newEvent");
    model.addAttribute(eventList);
    return "list";
}
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    Event event = new Event();
    event.setName("jipark");
    event.setId(10);
    this.mockMvc.perform(get("/events/list")
            .sessionAttr("visitTime", LocalDateTime.now())
            .flashAttr("newEvent", event))
            .andDo(print())
            .andExpect(status().isOk());
}
```

* RedirectAttributes의 `addAttribute()` 는 URL 파라미터에 붙을 수 있어야하기 때문에 문자열 형태만 가능하다는 제약이 있으나, FlashAttributes는 객체 자체도 전달이 가능하다.
* 주로 리다이렉트시에 데이터를 전달할 때 사용한다.
* 데이터가 URI에 노출되지 않는다.
* 임의의 객체를 저장할 수 있다.
* 보통 HTTP 세션을 사용한다.
* 리다이렉트 하기 전에 데이터를 HTTP 세션에 저장하고, 리다이렉트 요청을 처리 한 다음 그 즉시 제거한다.
* RedirectAttributes를 통해 사용할 수 있다.
* @ModelAttribute 대신 Model 객체에서 받을 수 있다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
