---
title: "[Spring Web MVC] 15장 : Header & Parameter"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-15T08:06:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. Header & Parameter

> SampleController.java

```java
@Controller
public class SampleController {

    @GetMapping(value = "/hello",
                headers = HttpHeaders.AUTHORIZATION + "=" + "111",
                params = "name=jipark")
    @ResponseBody
    public String hello() {
        return "hello";
    }
}
```

> SimpleControllerTest.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        this.mockMvc.perform(get("/hello")
                    .header(HttpHeaders.AUTHORIZATION, "111")
                    .param("name", "jipark"))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andExpect(content().string("hello"));
    }
}
```

* 특정한 헤더가 있는 요청을 처리하고 싶은 경우.
	* @RequestMapping(headers = “key”).
* 특정한 헤더가 없는 요청을 처리하고 싶은 경우.
	* @RequestMapping(headers = “!key”).
* 특정한 헤더 키/값이 있는 요청을 처리하고 싶은 경우.
	* @RequestMapping(headers = “key=value”).  
* 특정한 요청 매개변수 키를 가지고 있는 요청을 처리하고 싶은 경우.
	* @RequestMapping(params = “a”).
* 특정한 요청 매개변수가 없는 요청을 처리하고 싶은 경우.
	* @RequestMapping(params = “!a”).
* 특정한 요청 매개변수 키/값을 가지고 있는 요청을 처리하고 싶은 경우.
	* @RequestMapping(params = “a=b”).

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
