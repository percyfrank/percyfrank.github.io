---
title: "[Spring Web MVC] 22장 : RedirectAttributes"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:21:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. RedirectAttributes

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
    redirectAttributes.addAttribute("name", event.getName());
    redirectAttributes.addAttribute("id", event.getId());
    //redirect시 일반 model에 담은 상태라면 /events/list?name=xx&id=yy
    return "redirect:/events/list";
}

@GetMapping("/events/list")
public String getList(Model model,
                      @RequestParam String name,
                      @RequestParam Integer id,
                      @SessionAttribute LocalDateTime visitTime) {
    System.out.println(visitTime);
    Event event = new Event();
    List<Event> eventList = new ArrayList<>();
    event.setName("spring");
    eventList.add(event);
    model.addAttribute(eventList);
    return "list";
}

/*
* Alternative
* public String getList(@ModelAttribute("newEvent") Event event...)
*/
```

* 리다이렉트 할 때 기본적으로 Model에 들어있는 primitive type 데이터는 URI 쿼리 매개변수에 추가된다.
* Spring Boot에서는 이 기능이 기본적으로 비활성화 되어 있다.
  * Ignore-default-model-on-redirect 프로퍼티를 사용해서 활성화 할 수 있다.
* 원하는 값만 리다이렉트 할 때 전달하고 싶다면 RedirectAttributes에 명시적으로 추가할 수 있다.
  * 이 또한 URI 쿼리 매개변수에 추가된다.
  * 이러한 단점을 보완하기 위해 FlashAttributes를 사용할 수 있다.
* 리다이렉트 요청을 처리하는 곳에서 쿼리 매개변수를 @RequestParam 또는 @ModelAttribute로 받을 수 있다.
* @SessionAttributes와 @ModelAttribute가 중복된 value라면 @SessionAttributes를 먼저 참조한다.
  * @SessionAttributes에 event가 명시되어 있으며, @ModelAttribute는 @ModelAttribute("event")와 동일하다.
  * 위 예제는 Session이 소멸된 상태이기 때문에 바인딩 에러가 난다.
  * 따라서 @ModelAttribute로 바인딩할 시 **("newEvent")** 같이 value를 별도로 지정해야 에러가 나지 않는다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
