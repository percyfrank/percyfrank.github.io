---
title: "[Spring Web MVC] 10장 : MessageConverter"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-07T08:01:00-05:00
---

## 1. HTTP Message Converter

> SimpleController.java

```java
@GetMapping("/message")
@ResponseBody
public String message(@RequestBody String body) {
	 return body;
}
```

> SimpleControllerTest.java

```java
@Test
public void message() throws Exception {
	this.mockMvc.perform(get("/message").content("hello"))
		.andDo(print())
		.andExpect(status().isOk())
		.andExpect(content().string("hello"));
}
```

* 요청 본문에서 메시지를 읽어들이거나(@RequestBody), 응답 본문에 메시지를 작성할 때(@ResponseBody) 사용한다.
* @RestController를 사용하면 @ResponseBody를 생략할 수 있다.
* 요청 본문의 메시지를 특정 자료형으로 Conversion하며, 다시 응답의 Body에 담아 보내는 일련의 로직이다.
* Request의 Header 타입에 따라 어떤 컨버터를 쓸 지 결정된다.
* Postman 등으로도 테스트가 가능하다.

<br>

## 2. 기본 HTTP Message Converter

* 바이트 배열 컨버터.
* 문자열 컨버터.
* Resource 컨버터.
* Form 컨버터 (Form 데이터 to/from MultiValueMap<String, String>).
* (JAXB2 컨버터).
* (Jackson2 컨버터).
* (Jackson 컨버터).
* (Gson 컨버터).
* (Atom 컨버터).
* (RSS 컨버터).
* ...

<br>

## 3. 설정 방법

* 기본으로 등록해주는 컨버터 목록에 새로운 컨버터 추가하기 : extendMessageConverters.
* 기본으로 등록해주는 컨버터 목록은 다 무시하고 새로 컨버터 설정하기 : configureMessageConverters.
* 의존성 추가로 컨버터 등록하기. (권장)
	* 메이븐 또는 그래들 설정에 의존성을 추가하면 그에 따른 컨버터가 자동으로 등록 된다.
	* WebMvcConfigurationSupport.
	* 이 기능 자체는 스프링 프레임워크의 기능이다.

<br>

## 4. JSON Converter

> SimpleController.java

```java
@GetMapping("/jsonmessage")
@ResponseBody
public Person jsonMessage(@RequestBody Person person) {
	 return person;
}
```

> SimpleControllerTest.java

```java
@Autowired
ObjectMapper objectMapper;

@Test
public void jsonMessage() throws Exception {
	Person person = new Person();
	person.setId(2019l);
	person.setName("jipark");
	String jsonString = objectMapper.writeValueAsString(person);
	this.mockMvc.perform(get("/jsonmessage")
		.contentType(MediaType.APPLICATION_JSON)
		.accept(MediaType.APPLICATION_JSON)
		.content(jsonString))
		.andDo(print())
		.andExpect(status().isOk())
		.andExpect(jsonPath("$.id").value(2019))
		.andExpect(jsonPath("$.name").value(jipark));
}
```

* Spring Boot를 사용하지 않는 경우.
	* 사용하고 싶은 JSON 라이브러리를 의존성으로 추가한다.
	* GSON.
	* JacksonJSON.
	* JacksonJSON 2.
* Spring Boot를 사용하는 경우 기본적으로 JacksonJSON 2가 의존성에 들어있다.
* JSON path로 값의 비교가 더욱 명확하게 가능하다.
* ObjectMapper로 객체를 JSON String으로 쉽게 변환이 가능하다.

<br>

## 5. XML Converter

> pom.xml

```xml
<dependency>
	<groupId>javax.xml.bind</groupId>
	<artifactId>jaxb-api</artifactId>
</dependency>
<dependency>
	<groupId>org.glassfish.jaxb</groupId>
	<artifactId>jaxb-runtime</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-oxm</artifactId>
	<version>${spring-framework.version}</version>
</dependency>
```

> WebConfig.java

```java
@Bean
public Jaxb2Marshaller marshaller() {
	Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
	jaxb2Marshaller.setPackagesToScan(Person.class.getPackageNames());
	return jaxb2Marshaller;
}
```

> Person.java

```java
@XmlRootElement @Entity
```

<br>

> SimpleControllerTest.java

```java
@Autowired
Marshaller marshaller;

@Test
public void xmlMessage() throws Exception {
	Person person = new Person();
	person.setId(2019l);
	person.setName("jipark");
	StringWriter stringWriter = new StringWriter();
	Result result = new StreamResult(stringWriter);
	marshaller.marshal(person, result);
	String xmlString = stringWriter.toString();
	this.mockMvc.perform(get("/jsonmessage")
		.contentType(MediaType.APPLICATION_XML)
		.accept(MediaType.APPLICATION_XML)
		.content(xmlString))
		.andDo(print())
		.andExpect(status().isOk())
		.andExpect(xpath("person/name").string("jipark"));
}
```

* Spring Boot는 기본적으로 XML 의존성을 추가해주지 않는다.
* OXM(Object-XML Mapper) 라이브러리 중 Spring이 지원하는 JacksonXML 및 JAXB 등의 의존성을 추가한다.
* 특정 Root Element를 인지해야 XML 변환이 가능하다.
* JSON 예시처럼 헤더 정보를 그에 맞게 명시적으로 수정하고. Path를 확인할 수 있다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
