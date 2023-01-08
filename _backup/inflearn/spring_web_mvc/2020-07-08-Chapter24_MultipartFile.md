---
title: "[Spring Web MVC] 24장 : MultipartFile"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-08T10:23:00-05:00
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/inflearn-mvc)

## 1. MultipartFile

> FileController.java

```java
@Controller
public class FileController {

    @PostMapping("/file")
    public String fileUpload(@RequestParam("file") MultipartFile file,
                              RedirectAttributes attributes) {
        //save
        String message = file.getOriginalFilename() + "is successfully uploaded.";
        attributes.addFlashAttribute("message", message);
        return "redirect:/file";
    }

    @GetMapping("/file")
    public String fileUploadForm(Model model) {
        //@ModelAttribute를 사용하지 않아도 Model 객체에 리다이렉트 정보가 자동 저장됨
        return "/file/index";
    }
}
```

> index.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div th:if="${message}">
    <h2 th:text="${message}"/>
</div>
<form method="POST" enctype="multipart/form-data" action="#" th:action="@{/file}">
    File: <input type="file" name="file"/>
    <input type="submit" value="Upload"/>
</form>
</body>
</html>
```

> SampleControllerTest.java

```java
@Test
public void test() throws Exception {
    MockMultipartFile file = new MockMultipartFile("file",
            "test.txt",
            "text/plain",
            "hello file".getBytes());
    this.mockMvc.perform(multipart("/file")
            .file(file))
            .andDo(print())
            .andExpect(status().is3xxRedirection());
}
```

* 파일 업로드시 사용하는 메소드 아규먼트이다.
* MultipartResolver Bean이 설정 되어 있어야 사용할 수 있으나, Spring Boot는 자동 설정을 지원한다.
* POST multipart/form-data 요청에 들어있는 파일을 참조할 수 있다.
* List\<MultipartFile\> 아규먼트로 여러 파일을 참조할 수도 있다.
* 관련 설정은 MultipartAutoConfiguration 및 MultipartProperties 등이 있다.

<br>

## 2. ResponseEntity를 통한 다운로드

> FileController.java

```java
@Autowired
ResourceLoader resourceLoader;

@GetMapping("/file/{fileName}")
public ResponseEntity<Resource> fileDownload(@PathVariable String fileName) throws IOException {
    Resource resource = resourceLoader.getResource("classpath:" + fileName);
    File file = resource.getFile();
    Tika tika = new Tika();
    String type = tika.detect(file);
    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachement; filename=\"" +
            resource.getFilename() + "\"")
            .header(HttpHeaders.CONTENT_TYPE, type)
            .header(HttpHeaders.CONTENT_LENGTH, String.valueOf(file.length()))
            .body(resource);
}
```

* 응답 상태 코드와 응답 헤더 및 응답 본문 등을 설정할 수 있는 리턴 타입이다.
* Content-Disposition : 사용자가 해당 파일을 받을 때 사용할 파일 이름이다.
* Content-Type : 파일의 미디어 타입을 의미한다.
  * Tika 라이브러리를 통해 자동으로 파일 미디어 타입을 탐지한다.
* Content-Length : 파일의 용량을 의미한다.
* @ResponseBody를 붙여도 상관없으며, 붙이지 않아도 ResponseEntity가 응답임을 의미한다.

<br>

---

## References

*	스프링 웹 MVC(백기선, Inflearn)
