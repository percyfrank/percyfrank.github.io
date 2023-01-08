---
title: "[Spring Web MVC] 19장 : @RequestParam"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-14T09:10:00-05:00
---

## 1. @RequestParam

> SampleController.java

```java
@Controller
public class SampleController {

    @GetMapping("/events")
    @ResponseBody
    public Event event(@RequestParam Map<String, String> map) {
        Event event = new Event();
        event.setName(map.get("name"));
        return event;
    }
    /*
    * 가능한 표현
    * @RequestParam String name
    * @RequestParam(required = false, defaultValue = "jipark") String name
    * @RequestParam(value = "name", required = false, defaultValue = "jipark") String nameValue
    */
}
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    this.mockMvc.perform(get("/events").param("name", "jipark"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("name".value("jipark")));
}
```

* 요청 매개변수에 들어있는 단순 타입 데이터를 메소드 아규먼트로 받아올 수 있다.
  * GetMapping의 경우 /hello?name=jipark 형태로 들어온다.
* 값이 반드시 있어야 한다.
	* required=false 또는 Optional을 사용해서 부가적인 값으로 설정할 수도 있다.
* String이 아닌 값들은 타입 컨버전을 지원한다.
* Map\<String, String\> 또는 MultiValueMap\<String, String\>에 사용해서 모든 요청 매개변수를 받아 올 수도 있다.
* 애노테이션 생략이 가능하다.

<br>

## 2. 요청 매개변수

* 쿼리 매개변수
* 폼 데이터

<br>

## 3. Form Submit

> SampleController.java

```java
@Controller
public class SampleController {

    @GetMapping("/events/form")
    public String eventsForm(Model model) {
        model.addAttribute("event", new Event());
        return "form";
    }

    @PostMapping("/events")
    @ResponseBody
    public Event event(@RequestParam String name, @RequestParam Integer id) {
        Event event = new Event();
        event.setName(name);
        event.setId(id);
        return event;
    }
}
```

* GET : /events/form.
* 뷰: form.html.
* 모델 : “event”, new Event().

<br>

## 3. Thymeleaf

> form.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="#" th:action="@{/events}" method="post" th:object="${event}">
    <input type="text" title="name" th:field="*{name}"/>
    <input type="text" title="id" th:field="*{id}"/>
    <input type="submit" value="create"/>
</form>
</body>
</html>
```

> SampleControllerTest.java

```java
@Test
public void eventTest() throws Exception {
    this.mockMvc.perform(get("/events/form"))
            .andDo(print())
            .andExpect(view().name("form"))
            .andExpect(model().attributeExists("event"))
            .andExpect(status().isOk());
}
```

*	@{} : URL 표현식.
*	${} : variable 표현식.
*	\*{} : selection 표현식으로 기본값을 보여준다.
*	타임리프는 서블릿 컨테이너 엔진이 없어도 서버 사이드에서 렌더링이 끝나기 때문에, JSP와 다르게 View 결과 단위 테스트가 가능하다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
