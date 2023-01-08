---
title: "[번역] WebTestClient Reference 및 테스트 예제"
excerpt: "WebFlux의 WebTestClient를 사용해보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-08-26
last_modified_at: 2021-08-26
---

## 1. 서론

[WebTestClient Version 5.2.9.RELEASE](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/testing-webtestclient.pdf) 레퍼런스를 번역하고 테스트 예제를 간단하게 정리해보았다.

***오역이 많으며 Kotlin 코드 예제는 생략했습니다. 영어 원문은 해당 링크를 참조하길 바랍니다.***

<br>

## 2. WebTestClient

WebTestClient는 WebClient를 감싸고 있는 얇은 층으로, [WebClient](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-client)를 이용하여 요청을 수행하며 응답을 검증하기 위한 효과적인 API를 제공합니다. WebTestClient는 [Mock Request 및 Response](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/testing.pdf#mock-objects-web-reactive)를 사용하여 WebFlux 어플리케이션에 바인딩하며, HTTP 웹 서버를 테스트할 수 있습니다.

<br>

## 3. Setup

WebTestClient를 생성하기 위해, 여러가지 서버 설정 옵션들 중 하나를 선택하시길 바랍니다. 작동 중인 서버에 바인딩할 WebFlux 어플리케이션을 설정하거나, URL을 사용하여 직접 연결하십시오.

### 3.1. Bind to Controller

다음은 한 번에 하나의 @Controller를 테스트 하기 위한 서버 설정을 생성하는 예제입니다.

> Java

```java
client = WebTestClient.bindToController(new TestController()).build();
```

위의 예제는 [WebFlux Java configuration](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-config)을 로드하고 인자로 주어진 컨트롤러를 등록합니다. 이 경우, WebFlux 어플리케이션은 Mock Request 및 Response 객체를 사용하여 HTTP 서버 없이 테스트됩니다. builder에는 기본 WebFlux Java configuration을 커스터마이징 할 수 있는 다양한 메소드들이 있습니다.

### 3.2. Bind to Router Function

다음은 [RouterFunction](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-fn)을 통해 서버를 설정하는 예제입니다.

> Java

```java
RouterFunction<?> route = ...
client = WebTestClient.bindToRouterFunction(route).build();
```

내부적으로 해당 설정이 RouterFunctions.toWebHandler에 전달됩니다. 이 경우, WebFlux 어플리케이션은 Mock Request 및 Response 객체를 사용하여 HTTP 서버 없이 테스트됩니다.

### 3.3. Bind to ApplicationContext

다음은 어플리케이션의 Spring Configuration을 통해 서버를 설정하는 예제입니다.

> Java

```java
@SpringJUnitConfig(WebConfig.class) ①
class MyTests {

    WebTestClient client;

    @BeforeEach
    void setUp(ApplicationContext context) { ②
        client = WebTestClient.bindToApplicationContext(context).build(); ③
    }
}
```

① 로드할 configuration 클래스를 명시합니다.<br>
② 해당 설정을 주입합니다.<br>
③ WebTestClient를 생성합니다.

내부적으로 해당 설정이 WebHttpHandlerBuilder로 전달되어 요청 처리 체인을 설정합니다. 세부 내용은 [WebHandler API](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-web-handler-api)를 참조하십시오. 이 경우, WebFlux 어플리케이션은 Mock Request 및 Response 객체를 사용하여 HTTP 서버 없이 테스트됩니다.

### 3.4. Bind to Server

다음은 서버 설정 방법을 통해 작동중인 서버로 직접 연결하는 예제입니다.

> Java

```java
client = WebTestClient.bindToServer().baseUrl("http://localhost:8080").build();
```

### 3.5. Client Builder

상기한 서버 설정 옵션들과 더불어서, 베이스 URL과 기본 헤더 및 클라이언트 필터 등 다양한 클라이언트 옵션을 설정할 수 있습니다. 다음과 같이, bindToServer를 통해 이러한 옵션들을 쉽게 사용할 수 있습니다. 서버에서 클라이언트 설정으로의 이행을 위해 ``configureClient()``를 사용해야 합니다.

> Java

```java
client = WebTestClient.bindToController(new TestController())
        .configureClient()
        .baseUrl("/test")
        .build();
```

<br>

## 4. Writing Tests

WebTestClient는 [WebClient](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-client)와 동일한 API를 제공하며, ``exchange()`` 메소드를 사용하여 요청을 수행할 수 있습니다. 응답을 검증하기 위해, API는 ``exchange()`` 이후 체이닝 워크플로우를 지원합니다.

응답 상태 및 헤더에 대한 단정 테스트(Assertion)로 시작합니다.

> Java

```java
client.get().uri("/persons/1")
        .accept(MediaType.APPLICATION_JSON)
        .exchange()
        .expectStatus().isOk()
        .expectHeader().contentType(MediaType.APPLICATION_JSON)
```

그 다음, 응답 본문을 복호화 및 소비하는 방식을 명시해줍니다.

* ``expectBody(Class<T>)`` : 객체 하나로 복호화합니다.
* ``expectBodyList(Class<T>)`` : 복호화 후, List<T>로 객체들을 담습니다.
* ``expectBody()`` : 빈 본문 혹은 [JSON Content](https://xlffm3.github.io/spring%20&%20spring%20boot/WebTestClient/#42-json-content)을 byte[]로 디코딩합니다.

이후, 다음과 같이 본문 검증을 위해 내장된 단정 테스트(Assertion)을 사용할 수 있습니다.

> Java

```java
client.get().uri("/persons")
        .exchange()
        .expectStatus().isOk()
        .expectBodyList(Person.class).hasSize(3).contains(person);
```

다음과 같이, 내장된 Assertion을 넘어서 본인만의 Assertion을 이용해 상세하게 단정 테스트를 수행할 수 있습니다.

> Java

```java
import org.springframework.test.web.reactive.server.expectBody;

client.get().uri("/persons/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody(Person.class)
        .consumeWith(result -> {
        // custom assertions (e.g. AssertJ)...
        });
```

다음과 같이 API 워크 플로우(체이닝)을 빠져나와 결과를 얻을 수 있습니다.

> Java

```java
EntityExchangeResult<Person> result = client.get().uri("/persons/1")         
        .exchange()
        .expectStatus().isOk()
        .expectBody(Person.class)
        .returnResult();
```

응답 본문을 제너릭스를 사용하는 특정 클래스로 복호화하고자 한다면, Class<T>가 아닌 ParameterizedTypeReference<T> 를 허용하는 오버로드된 메소드를 사용하십시오.

### 4.1. No Content

만약 응답 본문에 내용이 없거나, 있더라도 신경쓰지 않을 경우 Void.class를 사용하십시오. 이를 통해 자원들이 해제되었음을 보장합니다. 예제는 다음과 같습니다.

> Java

```java
client.get().uri("/persons/123")
        .exchange()
        .expectStatus().isNotFound()
        .expectBody(Void.class);
```

만약 응답 본문 내용이 없다고 단정 테스트(Assertion)를 진행하고자 한다면, 다음과 같이 코드를 짤 수 있습니다.

> Java

```java
client.post().uri("/persons")
        .body(personMono, Person.class)
        .exchange()
        .expectStatus().isCreated()
        .expectBody().isEmpty();
```

### 4.2. JSON Content

``expectBody()``를 사용할 때, 응답은 byte[]로 소비됩니다. 이는 미가공 컨텐츠에 대한 단정 테스트(Assertion)를 수행하는데 유용합니다. 예를 들어, [JSONAssert](https://jsonassert.skyscreamer.org/)를 사용하여 다음과 같이 JSON 컨텐츠를 검증할 수 있습니다.

> Java

```java
client.get().uri("/persons/1")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .json("{\"name\":\"Jane\"}")
```

다음과 같이 [JSONPath](https://github.com/json-path/JsonPath) 표현을 사용할 수 있습니다.

> Java

```java
client.get().uri("/persons")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$[0].name").isEqualTo("Jane")
        .jsonPath("$[1].name").isEqualTo("Jason");
```

### 4.3. Streaming Response

"text/event-stream" 혹은 "application/stream+json"과 같이 무한한 스트림을 테스트할 때, 다음과 같이 응답 상태 및 헤더 단정 테스트(Assertion)를 수행한 다음 ``returnResult()``를 사용하여 API 워크 플로우(체이닝)을 종료해야 합니다.

> Java

```java
FluxExchangeResult<MyEvent> result = client.get().uri("/events")
        .accept(TEXT_EVENT_STREAM)
        .exchange()
        .expectStatus().isOk()
        .returnResult(MyEvent.class);
```

이제 Flux<T> 소비 및 복호화 된 객체들에 대해 단정 테스트를 진행할 수 있습니다. 또한 테스트 목적이 달성된 시점에 취소할 수 있습니다. reactor-test 모듈의 StepVerifier를 사용할 것을 권장합니다.

> Java

```java
Flux<Event> eventFlux = result.getResponseBody();

StepVerifier.create(eventFlux)
        .expectNext(person)
        .expectNextCount(4)
        .consumeNextWith(p -> ...)
        .thenCancel()
        .verify();
```

### 4.4 Request Body

요청 생성에 대해, WebTestClient는 WebClient와 동일한 API를 제공하며 실행 방식이 대체적으로 간단합니다. 폼 데이터 및 멀티파트 요청 등 본문이 있는 요청을 준비하기 위해, [WebClient documentation](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-client-body)를 참조하십시오.

<br>

## 5. WebClient Documentation 발췌

WebTestClient Documentation은 테스트 관련 여러 메소드에 대한 예제가 부족하다고 생각되어, WebClient Documentation 내용을 추가로 발췌했습니다.

### 5.1. retrieve()

``retrieve()`` 메소드는 응답 본문을 받고 복호화하는 가장 쉬운 방법입니다.

> Java

```java
WebClient client = WebClient.create("https://example.org");

Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .bodyToMono(Person.class);
```

다음과 같이, 응답으로부터 복호화한 객체들의 스트림을 받을 수 있습니다.

> Java

```java
Flux<Quote> result = client.get()
        .uri("/quotes").accept(MediaType.TEXT_EVENT_STREAM)
        .retrieve()
        .bodyToFlux(Quote.class);
```

기본적으로, 4xx 및 5xx 등의 응답 상태 코드들은  WebClientResponseException 혹은 WebClientResponseException.BadRequest 및 WebClientResponseException.NotFound와 같은 HTTP 상태 관련 서브 예외들을 호출합니다. 다음과 같이 ``onStatus()`` 메소드를 사용하여 결과 예외를 커스터마이징할 수 있습니다.

> Java

```java
Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> ...)
        .onStatus(HttpStatus::is5xxServerError, response -> ...)
        .bodyToMono(Person.class);
```

``onStatus()`` 메소드를 사용할 때, 응답이 본문 내용을 포함할 것으로 예상된다면, onStatus 콜백 메소드가 이를 소비합니다. 만약 아니라면, 본문 내용은 자동적으로 사라지며 자원들이 해제되었음을 보장합니다.

### 5.2. exchange()

``exchange()`` 메소드는 ``retrieve()`` 메소드보다 더 많은 제어권을 제공합니다. 다음의 예제는 ``retrieve()``와 동일하지만, ClientResponse에 대한 접근을 제공한다는 점에서 차이가 있습니다.

> Java

```java
Mono<Person> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .exchange()
        .flatMap(response -> response.bodyToMono(Person.class));
```

또한 현 단계에서, ResponseEntity를 생성가능합니다.

> Java

```java
Mono<ResponseEntity<Person>> result = client.get()
        .uri("/persons/{id}", id).accept(MediaType.APPLICATION_JSON)
        .exchange()
        .flatMap(response -> response.toEntity(Person.class));
```

``exchange()``는 ``retrieve()``와 다르게, 4xx 혹은 5xx 응답에 대해 자동 에러 시그널을 지원하지 않습니다. 상태 코드를 확인하고 어떻게 이를 처리할지 결정해야 합니다.

``exchange()``를 사용하는 경우, 성공과 실패 및 예외 데이터 등 시나리오에 상관없이 어플리케이션이 전적으로 응답 소비를 책임져야 합니다. 그렇지 않으면 메모리 누수를 유발할 수 있습니다. ClientResponse에 대한 Javadoc에서는 본문을 소비할 수 있는 모든 가용 옵션들을 설명합니다. ``exchange()``는 응답을 소비하기 전에 응답 상태 및 헤더를 항상 체크해야 하기 때문에, 특별한 이유가 없다면 일반적으로 ``retrieve()`` 사용을 권장합니다.

### 5.3. Request Body

요청 본문은 Mono처럼 ReactiveAdapterRegistry가 다루는 비동기 타입으로부터 부호화 될 수 있습니다. 예제는 다음과 같습니다.

> Java

```java
Mono<Person> personMono = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .body(personMono, Person.class)
        .retrieve()
        .bodyToMono(Void.class);
```

객체 스트림을 다음과 같이 부호화할 수 있습니다.

> Java

```java
Flux<Person> personFlux = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_STREAM_JSON)
        .body(personFlux, Person.class)
        .retrieve()
        .bodyToMono(Void.class);
```

만약 실제 값을 가지고 있다면, ``bodyValue()`` 라는 메소드를 사용할 수 있습니다.

> Java

```java
Person person = ... ;

Mono<Void> result = client.post()
        .uri("/persons/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(person)
        .retrieve()
        .bodyToMono(Void.class);
```

### 5.4. Form Data

폼 데이터를 보낼 때, 본문으로 MultiValueMap<String, String>를 제공할 수 있습니다. 본문 내용은 FormHttpMessageWriter에 의해 application/x-www-form-urlencoded로 자동 설정됩니다.

> Java

```java
MultiValueMap<String, String> formData = ... ;

Mono<Void> result = client.post()
        .uri("/path", id)
        .bodyValue(formData)
        .retrieve()
        .bodyToMono(Void.class);
```

다음과 BodyInserters를 사용하여, 폼 데이터를 인라인으로 제공할 수 있습니다.

> Java

```java
Mono<Void> result = client.post()
        .uri("/path", id)
        .body(BodyInserters.fromFormData("k1", "v1").with("k2", "v2"))
        .retrieve()
        .bodyToMono(Void.class);
```

### 5.5. Multipart Data

멀티파트 데이터를 보내기 위해, MultiValueMap<String, ?>를 제공해야 합니다. 값은 파트 내용을 담고 있는 인스턴스 혹은 파트에 대한 내용 및 헤더를 담고 있는 HttpEntity 인스턴스여야 합니다. MultipartBodyBuilder는 멀티파트 요청을 준비하는데 편리한 API를 제공합니다.

> Java

```java
MultipartBodyBuilder builder = new MultipartBodyBuilder();

builder.part("fieldPart", "fieldValue");
builder.part("filePart1", new FileSystemResource("...logo.png"));
builder.part("jsonPart", new Person("Jason"));
builder.part("myPart", part); // Part from a server request

MultiValueMap<String, HttpEntity<?>> parts = builder.build();
```

대부분의 경우 각 파트에 대해 Content-Type을 명시할 필요는 없습니다. 직렬화를 위해 선택된 HttpMessageWrite(Resource의 경우 파일 확장자)에 기반하여 컨텐츠 타입이 자동적으로 결정되기 때문입니다. 필요하다면, 오버로드된 빌더 ``part()`` 메소드를 통해 각 파트에 사용하고자 하는 MediaType을 명시할 수 있습니다.

MultiValueMap이 준비되면, ``body()`` 메소드를 통해 WebClient에 이를 쉽게 전달할 수 있습니다.

> Java

```java
MultipartBodyBuilder builder = ...;

Mono<Void> result = client.post()
        .uri("/path", id)
        .body(builder.build())
        .retrieve()
        .bodyToMono(Void.class);
```

만약 MultiValueMap이 정규 폼 데이터를 표현할 수 있는 non-String 값(application/x-www-form-urlencoded)을 한 가지라도 포함하고 있다면, Content-Type을 Multipart/form-data로 설정해야 합니다. (HttpEntity 래퍼를 보장하는 MultipartBodyBuilder를 사용할 때.)

MultipartBodyBuilder의 대안으로, 다음과 같이 BodyInserters를 통해 멀티파트 컨텐츠를 인라인 스타일로 제공할 수 있습니다.

> Java

```java
Mono<Void> result = client.post()
        .uri("/path", id)
        .body(BodyInserters.fromMultipartData("fieldPart", "value").with("filePart", resource))
        .retrieve()
        .bodyToMono(Void.class);
```

<br>

## 6. 테스트 예제

Reference를 바탕으로 Form Data를 검증하는 간단한 테스트를 작성해보자.

> ArticleControllerTest.java

```java
@ExtendWith(SpringExtension.class)
@AutoConfigureWebTestClient ①
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ArticleControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    public void 게시글_생성() {
        webTestClient.method(HttpMethod.POST) ②
                .uri("/articles")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .body(BodyInserters
                        .fromFormData("name", "jipark")
                        .with("content", "test")) ③
                .exchange()
                .expectStatus()
                .is3xxRedirection()
                .expectBody()
                .consumeWith(result -> {
                    System.out.println(result.getResponseHeaders());
                    System.out.println(result.getRequestHeaders().getLocation());
                    System.out.println(result.getRequestHeaders().getLocation().getPath());
                }); ④
    }
}
```

① 해당 어노테이션이 WebTestClient Bean을 생성해준다.<br>
② ``post()`` 등의 메소드가 있지만, webTestClient가 메소드 체이닝을 지원하기 때문에 추후 중북되는 테스트 코드들을 하나의 메소드로 추출하여 리팩토링하기 수월하도록 파라미터를 받는 ``method()``를 사용했다.<br>
③ BodyInserters를 통해 Form 혹은 Multipart Data를 인라인으로 Param을 보낸다.<br>
④ ``consumeWith()`` 메소드에서 세부적인 테스트가 가능하다. 요청이나 응답의 헤더와 본문 및 내용에 관련된 정보를 얻을 수 있다.

<br>

---

## References

* [WebTestClient Version 5.2.9.RELEASE](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/testing-webtestclient.pdf)
* [Web on Reactive Stack Version 5.2.9.RELEASE](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/pdf/web-reactive.pdf#webflux-client-body)
