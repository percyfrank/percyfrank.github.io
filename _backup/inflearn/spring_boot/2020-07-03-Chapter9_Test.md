---
title:  "[Spring Boot 개념과 활용] 9장 : Test"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-11T08:11:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-boot)

## 1. @SpringBootTest

* 별도의 Bean 설정 파일이 필요 없다.
  * 다만 JUnit4를 사용한다면 ``@RunWith(SpringRunner.class)``를 추가해야 한다.
* @SpringBootConfiguration이 자동으로 주요 설정 파일을 찾아주기 때문에, 별도의 @ContextConfiguration 지정이 필요 없다.
  * 테스트에 특정 설정 파일을 지정하려면 @TestConfiguration를 사용한다.
* @SpringBootTest는 통합 테스트이기 때문에 모든 Bean을 등록한다.
* webEnvironment.
  * MOCK(Default) : mock servlet environment이며, 내장 톰캣을 사용하지 않는다.
    * MOCK 베이스 기반으로 테스트를 진행하려면 @AutoConfigureMockMvc 혹은 @AutoConfigureWebTestClient 를 추가한다.
  * RANDON_PORT, DEFINED_PORT : 내장 톰캣을 사용한다.
  * NONE : 서블릿 환경을 제공하지 않는다.
* 클래스에 @Transactional을 달면 매 테스트 메서드 수행 직후 롤백되지만, RANDOM_PORT 및 DEFINED_PORT는 롤백되지 않는다.
  * HTTP Client와 서버가 서로 다른 스레드, 즉 트랜잭션에서 수행되기 때문이다.

<br>

## 2. DI와 테스트

* DI를 사용하면 단위 테스트를 수행하기 쉬워진다.
  * IoC 컨테이너 없이 직접 new 연산자로 객체를 생성할 수도 있다.
  * Mock 테스트를 수행하기 수월하다.

<br>

## 3. Mock

> ControllerTest.java

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
class ControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void testHello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello"))
                .andDo(print());
    }
}
```

* Mock된 서블릿과 인터랙션을 하기 위해 MockMvc라는 클라이언트를 사용해야 한다.
* @AutoConfigureMockMvc을 사용하고 MockMvc를 주입 받는 방식이 간편하다.

<br>

## 4. RANDON_PORT

> ControllerTest.java

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @Test
    public void testHello() throws Exception {
        String result = testRestTemplate.getForObject("/hello", String.class);

        assertThat(result).isEqualTo("hello");
    }
}
```

* RANDOM_PORT를 사용하는 경우 실제 내부 톰캣을 구동시키기 때문에, TestRestTemplate이나 WebClient를 주입받아 테스트를 진행한다.

<br>

## 5. @MockBean

> Controller.java

```java
@RestController
class Controller {

    @Autowired
    Service service;

    @GetMapping("/hello")
    public String hello() {
        return "hello" + service.getName();
    }
}
```

> ControllerTest.java

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void testHello() throws Exception {
        when(mockSampleService.getName()).thenReturn("mock hello");
        String result = testRestTemplate.getForObject("/hello", String.class);

        assertThat(result).isEqualTo("mock hello");
    }
}
```

* ApplicationContext에 들어있는 Bean을 Mock으로 만든 객체로 교체 한다.
* 모든 @Test 마다 자동으로 리셋한다.
* 원본 소스의 controller는 service의 메소드를 호출하는데, 이 때 호출되는 service를 ApplicationContext의 진짜 Service를 사용하는 것이 아닌 mockSampleService로 교체한다.

<br>

## 6. WebTestClient

```java
@AutoConfigureWebTestClient
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class DemoApplicationTests {

    @Autowired
    WebTestClient webTestClient;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void testHello() throws Exception {
        when(mockSampleService.getName()).thenReturn("mock hello");
        webTestClient.get().uri("/hello").exchange()
                .expectStatus().isOk()
                .expectBody(String.class).isEqualTo("mock hello");
    }
}
```

* Spring 5에 추가된 기능으로, WebFlux 의존성을 주입받아야 사용이 가능하다.
* WebClient는 Async하기 때문에, Test Code에서도 웹 클라이언트와 동일한 API를 사용할 수 있다.
  * 테스트 요청을 보내고 기다리는 것이 아니라, 응답이 오면 적절한 콜백을 확인할 수 있다.

<br>

## 7. 슬라이스 테스트

* @SpringBootTest는 @SpringBootApplication을 찾아 모든 Bean들을 등록하며 ApplicationContext를 생성한다.
* 통합 테스트가 비효율적이라면, 레이어 별로 잘라서 테스트하고 싶을 때 사용한다.
  * @JsonTest
  * @WebMvcTest
    * Web과 관련된 Bean들만 자동 등록하기 때문에, Service나 Repository 등의 Bean이 등록되지 않아 의존성이 끊길 수 있다.
    * 따라서 MockMvc을 사용할 때, @MockBean으로 테스트 대상이 사용하는 의존 객체들을 생성해야 한다.
    * 사용 예
      * ``@WebMvcTest(TargetController.class)``
  * @WebFluxTest
  * @DataJpaTest
  * ..

### 7.1. @WebMvcTest 예제 코드

> ControllerTest.java

```java
@WebMvcTest(Controller.class)
class ControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    SampleService mockSampleService;

    @Test
    public void testHello() throws Exception {
        //test 내용
    }
}
```

<br>

## 8. 테스트 유틸리티

> ControllerTest.java

```java
@ExtendWith(OutputCaptureExtension.class)
@SpringBootTest
class LogComponentTest {

    @Autowired
    private LogComponent logComponent;

    @Test
    void capture(CapturedOutput capturedOutput) {
        logComponent.printLog();

        assertThat(capturedOutput).contains("findable");
    }
}
```

* OutputCapture.
  * Log를 비롯해서 콘솔에 찍히는 모든 것을 캡쳐하는 유틸리티이다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
