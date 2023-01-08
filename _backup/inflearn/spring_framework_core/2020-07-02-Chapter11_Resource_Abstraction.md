---
title:  "[Spring Framework 핵심 기술] 11장 : Resource 추상화"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:16:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. Resource Abstraction

* org.springframework.core.io.Resource.
* java.net.URL을 추상화 한 것이며, Spring 내부에서 많이 사용하는 인터페이스이다.
* 추상화 이유?
	* 클래스패스 기준으로 리소스 읽어오는 기능이 없다.
	* ServletContext를 기준으로 상대 경로로 읽어오는 기능이 없다.
	* 새로운 핸들러를 등록하여 특별한 URL 접미사를 만들어 사용할 수는 있지만 구현이 복잡하고 편의성 메소드가 부족하다.
* Spring의 Resource 인터페이스는 로우 레벨 리소스에 대한 접근을 추상화하고 있다.
  * InputStreamSource를 확장한다.

<br>

## 2. Resource 읽어오기

```java
public class AppRunner implements ApplicationRunner {

    @Autowired
    ApplicationContext resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(resourceLoader.getClass());
				//WebServerApplicationContext
        Resource resource = resourceLoader.getResource("classpath:/test.txt");
        System.out.println(resource.getClass());
				//ClassPathResource
        Resource resource2 = resourceLoader.getResource("test.txt");
        System.out.println(resource2.getClass());
				//ServletContextResource
		}
}
```

* ResourceLoader 인터페이스를 통해 Resource를 획득할 수 있다.
  * Resource의 타입은 location 문자열과 ApplicationContext의 타입에 따라 결정 된다.
    * ClassPathXmlApplicationContext -> ClassPathResource.
    * FileSystemXmlApplicationContext -> FileSystemResource.
    * WebApplicationContext -> ServletContextResource.
* ApplicationContext의 타입에 상관없이 리소스 타입을 강제하려면 java.net.URL 접두어(+ classpath:) 중 하나를 사용할 수 있다.
  * classpath:/me/whiteship/config.xml -> ClassPathResource.
  * file:///some/resource/path/config.xml -> FileSystemResource.

> MyBean.java

```java
@Component
public class MyBean {

    private final Resource template;

    public MyBean(@Value("${template.path}") Resource template) {
        this.template = template;
    }

    // ...
}
```

* Property로 정의된 파일 경로를 @Value에 명시하여 Resource를 주입받을 수 있다.
  * 와일드카드를 사용하여 복수개의 Resource 배열을 받을 수 있다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
