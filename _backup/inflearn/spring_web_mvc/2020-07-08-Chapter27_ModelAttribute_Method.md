---
title: "[Spring Web MVC] 27장 : @ModelAttribute Method"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:26:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. @ModelAttribute의 또 다른 사용법

> SampleController.java

```java
@ModelAttribute
public void subjects(Model model) {
    model.addAttribute("categories", List.of("study", "seminar", "hobby", "social"));
}
//혹은
@ModelAttribute("categories")
public List<String> subjects() {
    return new ArrayList<>().of("study", "seminar", "hobby", "social");
}

@GetMapping("/events/form/name")
@ModelAttribute //생략 가능
public Event eventFormName() {
    return new Event();
    //RequestToViewNameTranslator 인터페이스가 요청 이름과 일치하는 뷰 이름으로 리턴을 해줌.
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
            .andExpect(model().attributeExists("categories"))
            .andExpect(status().isOk());
}
```

* @RequestMapping을 사용한 핸들러 메소드의 아규먼트에 사용한다.
* @Controller 또는 @ControllerAdvice를 사용한 클래스에서 Model 정보를 초기화 할 때 사용한다.
* @RequestMapping과 같이 사용하면 해당 메소드에서 리턴하는 객체를 Model에 넣어 준다.
	* RequestToViewNameTranslator.
* 다른 핸들러에서 공통으로 사용하는 Model을 처리하는데 유용하다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
