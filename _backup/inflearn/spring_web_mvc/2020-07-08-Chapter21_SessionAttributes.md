---
title: "[Spring Web MVC] 21장 : @SessionAttribute & @SessionAttributes"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:20:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. @SessionAttributes

> SampleController.java

```java
@Controller
@SessionAttributes({"event", "book"})
public class SampleController {

    @GetMapping("/events/form")
    public String eventsForm(Model model, SessionStatus sessionStatus) {
        model.addAttribute("event", new Event());
        sessionStatus.setComplete();
        return "form";
    }
}
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    this.mockMvc.perform(get("/events/form"))
                .andExpect(request().sessionAttribute("event", notNullValue()));
}
```

* HttpSession을 직접 사용할 수도 있지만, 애노테이션에 설정한 이름에 해당하는 모델 정보를 자동으로 세션에 넣어준다.
  * @SessionAttributes에 명시된 이름을 Model의 Key로 삽입할 때, 해당 Key-Value 정보가 자동으로 Session에 담긴다.
  * @ModelAttribute는 세션에 있는 데이터도 바인딩한다.
* 여러 화면(또는 요청)에서 사용해야 하는 객체를 공유할 때 사용한다.
* SessionStatus를 사용해서 세션 처리 완료를 알리며, 폼 처리가 끝나고 세션을 비울 때 사용한다.

<br>

## 2. Multi Form Submit

```java
@Controller
@SessionAttributes({"event", "book"})
public class SampleController {

    @GetMapping("/events/form/name")
    public String eventsForm(Model model) {
        model.addAttribute("event", new Event());
        return "form-name";
    }

    @PostMapping("/events/form/name")
    public String event(@Validated @ModelAttribute Event event, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "form-name";
        }
        return "redirect:/events/form/id";
    }

    @GetMapping("/events/form/id")
    public String id(@ModelAttribute Event event, Model model) {
        model.addAttribute("event", event);
        return "form-id";
    }

    @PostMapping("/events/form/id")
    public String eventId(@Validated @ModelAttribute Event event,
                          BindingResult bindingResult,
                          SessionStatus sessionStatus) {
        if (bindingResult.hasErrors()) {
            return "form-id";
        }
        sessionStatus.setComplete();
        return "redirect:/events/list";
    }

    @GetMapping("/events/list")
    public String getList(Model model) {
        Event event = new Event();
        List<Event> eventList = new ArrayList<>();
        event.setName("spring");
        eventList.add(event);
        model.addAttribute(eventList);
        return "list";
    }
}
```

<br>

## 3. @SessionAttribute

> WebConfig.java

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new VisitTimeInterceptor());
    }
}
```

> VisitTimeInterceptor.java

```java
public class VisitTimeInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        if (session.getAttribute("visitTime") == null) {
            session.setAttribute("visitTime", LocalDateTime.now());
        }
        return true;
    }
}
```

> SampleController.java

```java
@GetMapping("/events/list")
public String getList(Model model, @SessionAttribute LocalDateTime visitTime) {
    System.out.println(visitTime);
    /* Alternative Code
    *  LocalDateTime visitTime = (LocalDateTime)httpSession.getAttribute("visitTime");
    */
    return "list";
}
```

* HTTP 세션에 들어있는 값을 참조할 때 사용한다.
* HttpSession을 사용할 때 비해 타입 컨버전을 자동으로 지원하기 때문에 조금 더 편리하다.
* HTTP 세션에 데이터를 넣고 빼고 싶은 경우에는 HttpSession을 사용한다.
* @SessionAttributes는 해당 컨트롤러 내에서만 동작하며, 해당 컨트롤러 안에서 다루는 특정 모델 객체를 세션에 넣고 공유할 때 사용한다.
* @SessionAttribute는 컨트롤러 밖(인터셉터 또는 필터 등)에서 만들어 준 세션 데이터에 접근할 때 사용한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
