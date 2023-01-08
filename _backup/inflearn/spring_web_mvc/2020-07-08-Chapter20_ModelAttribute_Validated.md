---
title: "[Spring Web MVC] 20장 : @ModelAttribute & @Validated"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T09:20:00-05:00
---

## 1. @ModelAttribute

> Event.java

```java
public class Event {

    @Min(0)
    Integer id;
    String name;
}
```

> SampleController.java

```java
@PostMapping("/events")
@ResponseBody
public Event event(@Valid @ModelAttribute Event event, BindingResult bindingResult) {
    if (bindingResult.hasError()) {
      //생략
    }
    return event;
}
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    this.mockMvc.perform(post("/events")
            .param("name", "jipark")
            .param("id", "10"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("name").value("jipark"))
            .andExpect(jsonPath("id").value("10"));
}
```

* @RequestParam은 단순 타입을 바인딩한다.
* @ModelAttribute는 여러 곳에 있는 단순 타입 데이터를 복합 타입 객체로 받아오거나 해당 객체를 새로 만들 때 사용할 수 있다.
  * 여러 곳: URI 패스, 요청 매개변수, 세션 등을 의미한다.
* 생략이 가능하다.

<br>

## 2. BindException 및 @Valid

* 타입 불일치 등 바인딩이 불가능한 경우 BindException 및 400 에러가 발생한다.
* 바인딩 에러를 직접 다루고 싶은 경우, BindingResult 타입의 아규먼트를 바로 오른쪽에 추가한다.
* 바인딩 이후에 검증 작업을 추가로 하고 싶은 경우, @Valid 또는 @Validated 애노테이션을 사용한다.
* Id가 음수값인 경우 맵핑되어도 @Valid에서 걸린다.
  * @Min, @Max, @NotNull 등.

<br>

## 3. @Validated

> Event.java

```java
public class Event {

    interface ValidateName {}
    interface ValidateId {}

    @Min(value = 0, groups = ValidateId.class)
    Integer id;
}
```

> SampleController.java

```java
@PostMapping("/events")
@ResponseBody
public Event event(@Validated(Event.ValidateId.class) @ModelAttribute Event event, BindingResult bindingResult) {
    if (bindingResult.hasError()) {
      //생략
    }
    return event;
}
```

* 스프링 MVC 핸들러 메소드 아규먼트에 사용할 수 있으며, validation group이라는 힌트를 사용할 수 있다.
* @Validated는 스프링이 제공하는 애노테이션으로 그룹 클래스를 설정할 수 있다.
* @Valid 애노테이션에는 그룹을 지정할 방법이 없다.

<br>

## 4. Form Submit

> form.html

```html
<p th:if="${#fields.hasErrors('limit')}" th:errors="*{limit}">Incorrect date</p>
```

* 타임리프 자체적으로 바인딩 에러를 보여줄 수 있다.

> list.html

```html
<a th:href="@{/form}">Create New Event</a>
<div th:unless="${#lists.isEmpty(eventList)}">
    <ul th:each="event: ${eventList}">
        <p th:text="${event.name}">Event Name</p>
    </ul>
</div>
```

> SampleController.java

```java
@GetMapping("/form")
public String eventForm(Model model) {
    Event event = new Event();
    event.setLimit(50);
    model.addAttribute("event", event);
    return "form";
}

@PostMapping("/events")
public String createEvent(@Validated @ModelAttribute Event event, BindingResult bindingResult, Model model) {
    if (bindingResult.hasErrors()) {
        return "form";
    }
    //DB 저장하는 구간
    return "redirect:list";
}

@GetMapping("/list")
public String getEvent(Model model) {
    Event event = new Event();//DB에서 읽는 구간
    event.setName("spring");
    event.setId(10);
    List<Event> eventList = new ArrayList<>();
    eventList.add(event);
    model.addAttribute("eventList", eventList);
    return "list";
}
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    ResultAction resultAction = this.mockMvc.perform(post("/events")
            .param("name", "jipark")
            .param("id", "-10"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(model().hasErrors());
    ModelAndView mav = resultAction.andReturn().getModelAndView();
    Map<String, Object> model = mav.getModelAndView();
    System.out.println(model.size()); //2
}
```

* @Validated 등을 통해 바인딩 에러가 발생하면 Model에는 Event 및 BindingResult.event 정보가 담긴다.
* Post 이후에 F5 등 브라우저 리프레시를 시도하면 Post 재요청(중복 form submit)이 발생한다.
* 이를 해결하는 방안으로 Post / Redirect / Get 패턴을 이용한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
