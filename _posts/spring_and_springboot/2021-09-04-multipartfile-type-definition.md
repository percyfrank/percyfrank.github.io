---
title: "Spring Boot MultipartFile 전송시 발생하는 Type Definition 에러"
excerpt: "MessageConverter가 InputStream을 직렬화하지 못하는 이슈다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-09-04
last_modified_at: 2021-09-04
---

## 1. MultipartFile 전송 코드

> AcceptanceTest.java

```java
@DisplayName("게시물 작성")
protected ResponseSpec api_게시물_작성(String title, String content, String token) {
    List<MultipartFile> files = FileFactory.getSuccessImageFiles();
    MultiValueMap<String, Object> multiValueMap = new LinkedMultiValueMap<>();
    multiValueMap.add("title", title);
    multiValueMap.add("content", content);
    multiValueMap.add("files", files);
    return webTestClient.post()
        .uri("/api/posts")
        .headers(header -> header.setBearerAuth(token))
        .contentType(MediaType.MULTIPART_FORM_DATA)
        .accept(MediaType.APPLICATION_JSON)
        .bodyValue(multiValueMap)
        .exchange()
        .expectStatus()
        .isCreated();
}
```

> Log

```bash
org.springframework.core.codec.CodecException: Type definition error: [simple type, class java.io.ByteArrayInputStream]; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class java.io.ByteArrayInputStream and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: java.util.Arrays$ArrayList[0]->org.springframework.mock.web.MockMultipartFile["inputStream"])

```

WebTestClient 공식 문서를 참고하면서 MultipartFile을 전송하는 API 코드를 작성했으나, 위 에러가 발생했다. RestTemplate 혹은 WebClient의 기본 MessageConverters가 MultipartFile 파일에 포함 된 InputStream을 직렬화하는 방법을 알지 못하기 때문에 예외가 발생하는 것이라고 한다.

해결을 위해서는 request 전송시 MultipartFile 자체 대신 MultipartMap에 MultipartFile의 바이트를 추가하여 해결한다.

<br>

## 2. 해결

> AcceptanceTest.java

```java
@DisplayName("게시물 작성")
protected ResponseSpec api_게시물_작성(String title, String content, String token) {
    return webTestClient.post()
        .uri("/api/posts")
        .headers(header -> header.setBearerAuth(token))
        .contentType(MediaType.MULTIPART_FORM_DATA)
        .accept(MediaType.APPLICATION_JSON)
        .bodyValue(generateMultipartWithImages(title, content))
        .exchange()
        .expectStatus()
        .isCreated();
}

@DisplayName("멀티파트 Body (이미지 포함)")
public static MultiValueMap<String, Object> generateMultipartWithImages(
    String title,
    String content
) {
    List<MultipartFile> files = FileFactory.getSuccessImageFiles();
    MultiValueMap<String, Object> multiValueMap = new LinkedMultiValueMap<>();
    multiValueMap.add("title", title);
    multiValueMap.add("content", content);
    files.forEach(file -> {
        try {
            Resource resource =
                new FileSystemResource(file.getBytes(), file.getOriginalFilename());
            multiValueMap.add("files", resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
    });
    return multiValueMap;
}

public static class FileSystemResource extends ByteArrayResource {

    private String fileName;

    public FileSystemResource(byte[] byteArray, String filename) {
        super(byteArray);
        this.fileName = filename;
    }

    public String getFilename() {
        return fileName;
    }

    public void setFilename(String fileName) {
        this.fileName = fileName;
    }
}
```

* 코드를 조금 더 간소화하는 방법에 대해 고민해봐야겠다.

<br>

---

## References

* [Rest-Api Call Other Rest-Api In Spring boot Not Working for (Multipart-File)](https://stackoverflow.com/questions/52152337/rest-api-call-other-rest-api-in-spring-boot-not-working-for-multipart-file/52163729)
* [MultipartFile Restemplate 전송시 오류](https://lovia98.github.io/blog/file-http-convert/)
