---
title:  "[Spring Boot 개념과 활용] 25장 : Rest Client"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-13T08:15:00-05:00
---

## 1. Rest Template

> SampleController.java

```java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public String hello() throws InterruptedException {
        Thread.sleep(5000L);
        return "hello";
    }

    @GetMapping("/bye")
    public String bye() throws InterruptedException {
        Thread.sleep(3000L);
        return "bye";
    }
}
```

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    RestTemplateBuilder restTemplateBuilder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        RestTemplate restTemplate = restTemplateBuilder.build();
        StopWatch stopWatch = new StopWatch();

        stopWatch.start();
        String helloResult = restTemplate.getForObject("http://localhost:8081/hello", String.class);
        System.out.println(helloResult);

        String byeResult = restTemplate.getForObject("http://localhost:8081/bye", String.class);
        System.out.println(byeResult);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```

* Blocking I/O 기반의 Synchronous API.
* RestTemplateAutoConfiguration.
* 프로젝트에 spring-web 모듈이 있다면 RestTemplateBuilder를 Bean으로 등록해준다.
* 동기적으로 처리하기 때문에 hello로 get 요청이 가고 나서 처리가 되기 전 까지(즉, 5초) 다음 라인으로 넘어가지 않는다.
  * 최종적으로 prettyPrint 결과는 8초가 나온다.

<br>

## 2. WebClient

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    WebClient.Builder builder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        WebClient webClient = builder.build();
        StopWatch stopWatch = new StopWatch();

        stopWatch.start();
        Mono<String> helloMono = webClient.get().uri("http://localhost:8081/hello")
                .retrieve()
                .bodyToMono(String.class);
        helloMono.subscribe(s -> {
            System.out.println(s);

            if (stopWatch.isRunning()) {
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });

        Mono<String> byeMono = webClient.get().uri("http://localhost:8081/bye")
                .retrieve()
                .bodyToMono(String.class);
        byeMono.subscribe(s -> {
            System.out.println(s);

            if (stopWatch.isRunning()) {
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });
    }
}
```

* Non-Blocking I/O 기반의 Asynchronous API.
* WebClientAutoConfiguration.
* 프로젝트에 spring-webflux 모듈이 있다면 WebClient.Builder를 Bean으로 등록해준다.
* 의존성 : spring-boot-starter-webflux.
* Mono를 만들고 subscribe를 해야 실제 요청을 보내고 응답을 받앙온다.
  * 코드 자체는 Non-Blocking하기 때문에 bye가 3초 즈음 먼저 찍히고, hello가 5초 즈음 찍힌다.
  * 최종적으로 prettyPrint의 결과는 비동기적이기 때문에 5초가 된다.

<br>

## 3. 커스터마이징

> AppRunner.java

```java
//지엽적인 커스터마이징
WebClient webClient = builder
        .baseUrl("http://localhost:8081")
        .build();
Mono<String> byeMono = webClient.get().uri("/bye");
```

* builder 패턴을 통해 다양한 메소드로 쿠키 및 헤더 등 커스터마이징이 가능하다.

> Application.java

```java
@Bean
public WebClientCustomizer webClientCustomizer() {
    return new WebClientCustomizer() {
        @Override
        public void customize(WebClient.Builder webClientBuilder) {
            webClientBuilder.baseUrl("http://localhost:8081/");
        }
    };
}

@Bean
public RestTemplateCustomizer restTemplateCustomizer() {
    return new RestTemplateCustomizer() {
        @Override
        public void customize(RestTemplate restTemplate) {
            restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
        }
    };
}

//혹은 람다식으로 간단하게
@Bean
public WebClientCustomizer webClientCustomizer() {
    return webClientBuilder -> webClientBuilder.baseUrl("http://localhost:8081");
}

@Bean
public RestTemplateCustomizer restTemplateCustomizer() {
    return restTemplate -> restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
}
```

* RestTemplate
  * 기본으로 java.net.HttpURLConnection을 사용한다.
  * 글로벌 커스터마이징.
    * RestTemplateCustomizer.
    * Bean 재정의.
* WebClient
  * 기본으로 Reactor Netty의 HTTP 클라이언트를 사용한다.
  * 글로벌 커스터마이징.
    * WebClientCustomizer.
    * Bean 재정의.
* Bean으로 등록하면 모든 Builder들의 baseUrl이 위와 같은 상태로 주입된다.
* RestTemplate도 비슷한 방식으로 커스터마이징하며, 다른 WAS의 httpClient를 이용하는 예제이다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
