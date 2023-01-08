---
title:  "[Spring Framework 핵심 기술] 2장 : ApplicationContext 외 다양한 Bean 설정 방법"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-09T08:07:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-core)

## 1. 설정

* 설정 메타파일은 IoC Container가 Bean들을 어떻게 의존 관계를 설정하여 인스턴스화 할 것인지에 대한 명세다.
  * 설정 메타파일의 방식으로는 XML 파일, 애너테이션, Java 기반 등이 있다.
  * XML 파일 설정 방식은 [공식 문서]([https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-xml-import](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-xml-import))를 참고하자.
* ApplicationContext 구현체(IoC 컨테이너)를 통해 어떤 방식의 설정 메타파일을 사용할 것인지 지정할 수 있다.
  * XML : ClassPathXmlApplicationContext, FileSystemXmlApplication 등.
  * Java : AnnotationConfigApplicationContext.
* 설정 방식의 진화 과정?
  * 초기에는 별도의 XML 설정 파일을 통해 Bean을 등록했다.
  * 이후 XML 설정 파일에는 Component Scan을 적용할 패키지를 명시하고, @Component 애너테이션을 단 클래스를 스캔해 자동으로 Bean으로 등록했다.
    * 초기의 XML 설정 방식보다는 유연해졌으나 별도의 XML 파일이 필요하다는 점에서는 한계가 있다.
  * 이후 @Configuration 애너테이션을 단 클래스 파일에서 @Bean을 등록하는 AnnotationConfigApplicationContext 방식을 사용했다.
  * @ComponentScan 애너테이션은 특정 패키지나 클래스의 위치에서 Bean을 스캔하여 등록한다.

## 1.1. Container 사용

* ApplicationContext란 Bean 및 의존성 등을 등록하는 팩토리다.
* ``getBean()`` 메서드를 통해 등록된 Bean 인스턴스를 받아올 수 있다.
  * 도메인 등 어플리케이션 코드에서 해당 메서드를 사용하면 Spring APi와  의존성이 생기므로 사용을 최대한 지양한다.

<br>

## 2. 설정 예제

### 2.1. XML Configuration

> BookService.java

```java
public class BookService {

    private ookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

> Application.xml

```xml
<bean id="bookService" class="com.example.demo.BookService">
    <property name="bookRepository" ref="bookRepository"/>
</bean>
<bean id="bookRepository" class="com.example.demo.BookRepository"/>
```

> DemoApplication.java

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXMLApplicationContext("Application.xml");
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        System.out.println(Arrays.toString(beanDefinitionNames)); // [bookService, bookRepository]
        BookService bookService = (BookService)context.getBean("bookService");
        System.out.println(bookService.bookRepository != null); //true
    }
}
```
* XML 설정 파일을 통해 POJO 오브젝트들의 관계를 맺어 Bean으로 등록한다.
* property name에 해당하는 bookRepository는 setter 메소드의 인자를 의미한다.
* property ref에 해당하는 bookRepository는 Bean ID로서, 해당 Bean을 참조한다는 의미이다.
* 이런 설정을 통해, Bean으로 등록된 bookRepository를 가져와 setter 메소드에 주입한다.
* 설정이 번거롭기 때문에 잘 사용하지 않는 방법이다.

### 2.2. XML ComponentScan

> BookService.java

```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

> BookRepository.java

```java
@Repository
public class BookRepository {}
```

> Application.xml

```xml
<context:component-scan basepackage="com.example.demo"/>
```

* XML에 등록된 component-scan 기능을 토대로 특정 패키지 이하의 모든 클래스 중에 @Component Annotation을 이용한 Class를 스캐닝하여 Bean으로 등록한다.
* @Service 및 @Repository는 @Component를 확장한 Annotation이다.
* @Autowired를 통해 의존성을 주입한다.
* 그러나 Bean 설정을 XML이 아닌 Java로 관리하기 위한 목적으로 Java 설정 파일이 등장하게 되었다.

### 2.3. Java @Configuration

> ApplicationConfig.Java

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public BookRepository bookRepository() {
        return new BookRepository();
    }

    @Bean
    public BookService bookService() { // 혹은 bookService(BookRepository bookRepository)
        BookService bookService = new BookService();
        bookService.setBookRepository(bookRepository());
        return new BookService();
    }
}
```

> DemoApplication.java

```java
ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
context.getBean("bookService", BookService.class);
```

* @Configuration Annotation을 통해 Bean을 더욱 유연하게 설정할 수 있다.
  * 주로 @Configuration 클래스에 @Bean 메서드로 Bean 정의를 등록한다.
  * POJO 오브젝트들의 의존 관계를 @Configuration 클래스에 정의하고, @Bean 애너테이션으로 등록해둔다.
* @Configuration 클래스 내부에서는 Bean간의 상호 의존성을 @Bean 메서드 호출로 표현할 수 있다.
  * @Component 클래스에 정의된 @Bean 메서드는 불가능하다.
* ``bookService()`` 메소드 내부에서 setter를 통해 의존성을 직접 주입할 수 있으며, 혹은 메소드 파라미터를 사용해서 의존성을 주입할 수 있다.
* 위 예제의 경우 생성자가 없이 setter만 사용하는 간단한 예제이기 때문에, ``bookService()`` 메소드 내부에서 직접 의존성을 주입할 필요 없이, 단순히 Bean 등록을 하고 @Autowired를 사용해도 된다.

<br>

### 2.4. Java @ComponentScan

> ApplicationConfig.java

```java
@Configuration
@ComponentScan(basePackageClasses = DemoApplication.class)
//or
@ComponentScan(basePackages = "com.example.demo")
public class ApplicationConfig {
}
```

*	특정 패키지나 클래스가 있는 위치에서 @Component 애너테이션을 단 Bean을 스캔하여 등록하는 방법이다.
*	SpringBoot와 가장 유사한 방법이며, SpringBoot는 @SpringBootApplication을 이용한다.
  *	@SpringBootApplication은 @ComponentScan과 @Configuration을 모두 갖고 있기 때문에, ApplicationConfig 파일이 필요없다.
    * 즉, SpringBoot를 실행하면 자동으로 패키지 내부의 Bean들을 스캔해 등록한다.

> DemoApplication.java

```java
ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
context.getBean("bookService", BookService.class);
```

* 일반 Spring을 사용한다면 다른 방법과 유사하게 ApplicaitonContext 생성 시 Configuration 클래스 정보를 제공하여 스캔해 Bean을 가져온다.

<br>

---

## References

*	스프링 프레임워크 핵심 기술(백기선, Inflearn)
