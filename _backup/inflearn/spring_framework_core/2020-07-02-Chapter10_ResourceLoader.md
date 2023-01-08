---
title:  "[Spring Framework 핵심 기술] 10장 : ResourceLoader"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-10T08:15:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. ResourceLoader

> AppRunner.java

```java
@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:text.txt");
        System.out.println(resource.getDescription()); //class path resource [text.txt]
    }
}
```

* 리소스를 읽어오는 기능을 제공하는 인터페이스이다.
* ApplicationContext가 ResourceLoader 클래스를 상속받아 사용한다.
* Resource에 있는 파일들이 빌드할 때 target 디렉토리로 들어가며, classpath를 기준으로 리소스를 찾게 된다.
* 파일 시스템과 클래스 패스 및 URL과 절대/상대 경로 등을 통해 리소스를 읽을 수 있다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
