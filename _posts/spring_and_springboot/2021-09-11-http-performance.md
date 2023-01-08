---
title: "HTTP 서버 성능 개선"
excerpt: "리소스 압축 및 캐싱을 적용해보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-09-11
last_modified_at: 2021-09-11
---

## 1. HTTP 서버 성능 개선

웹 어플리케이션의 규모가 커지면 다운로드하는 데이터 양이 늘어나는 등 성능 저하 이슈가 발생하기 마련이다. 즉각적인 웹 환경을 구현하려면 수십 혹은 수백개의 리소스를 수백 밀리초 내에 가져와야 한다.

### 1.1. 콘텐츠 효율성 최적화

* 불필요한 다운로드 제거
* 리소스 최적화 및 압축
* HTTP 캐싱

<br>

## 2. 불필요한 다운로드 제거

가장 효율적인 바어은 불필요하고 사용하지 않을 리소스를 전송하지 않는 것이다.

* 팀원들과 정기적인 논의를 통해 전송할 필요가 없는 리소스를 제거한다.
* JS의 Webpack 코드 분할을 사용해 필요한 부분만 전송한다.
* CSS, JS 파일을 각각 하나로 만들어 압축해 배포한다.
* 이미지 아이콘 대신 CSS, SVG를 활용하는 방법도 있다.
* CSS Image Sprite와 같이 여러 개의 이미지를 하나의 이미지로 합쳐서 관리한다.

<br>

## 3. 리소스 최적화 및 압축

전송할 필요가 없는 리소스를 최대한 배제시켰다면, 그 이상으로 최적화할 수 있는 방법을 강구해보자.

### 3.1. Minifier

![image](https://user-images.githubusercontent.com/56240505/132950947-bbcc72fd-ca1d-4fd7-afe7-19ee4565183e.png)

* HTML, JS, CSS 등의 리소스 자체를 최적화한다.
* [HTML, JS, CSS Minifier](https://developers.google.com/speed/docs/insights/MinifyResources)는 코드 주석 및 형식 지정, 사용되지 않는 코드 삭제, 더 짧은 변수와 함수 이름 사용하기 등을 통해 정적 파일의 크기를 줄인다.
* 웹 페이지에서 수동으로 줄일 수 있으나 프론트 코드를 배포하기 전에 Compressor 하는 기능을 라이브러리로 자동화할 수 있을 것으로 보인다.

#### HTML

> Shell

```bash
$ npm install html-minifier -g
$ html-minifier --collapse-whitespace \
--remove-comments \
--remove-optional-tags \
--remove-redundant-attributes \
--remove-script-type-attributes \
--remove-tag-whitespace \
--use-short-doctype \
--minify-css true \
--minify-js true \
-o minified.html original.html
```

![image](https://user-images.githubusercontent.com/56240505/132951593-d1dae408-9d72-4e93-8205-54c2b44d6f60.png)

#### CSS

> Shell

```bash
$ npm i -g csso-cli
$ csso input.css --output output.css
```

![image](https://user-images.githubusercontent.com/56240505/132951901-9e55147a-54b8-476c-9573-07e89e3c4d54.png)

#### JS

> Shell

```bash
$ npm i -g uglify-js
$ uglifyjs -c -m -o output.js input.js
```

![image](https://user-images.githubusercontent.com/56240505/132951976-be240c71-db66-4b46-b996-b2731fd6da5c.png)

### 3.2. GZIP 압축

압축은 브라우저에서 리소스를 처리하는 방식에 영향을 주지 않고 불필요하거나 중복된 데이터를 삭제하는 것을 의미한다.

* GZIP은 모든 바이트 스트림에 적용할 수 있는 범용 압축 프로그램이다.
  * 텍스트 기반 리소스인 CSS, JS, HTML에서 최상의 성능을 낸다.
  * 모든 최신 브라우저는 GZIP 압축을 지원한다.
* Brotil을 통해 압축할 수 있지만, GZIP에 비해 모든 버전의 브라우저를 지원하지 않는 등 보편성이 떨어진다.

> application.properties

```properties
server.compression.enabled=true
```

* 압축이 수행되기 위해서는 응답이 최소 2048 바이트는 되어야 한다.
* 최소 사이즈는 ``server.compression.min-response-size`` 설정을 조절한다.

```
text/html
text/xml
text/plain
text/css
text/javascript
application/javascript
application/json
application/xml
```

* 기본적으로 위 MIME 컨텐츠 타입의 응답들이 압축되며, ``server.compression.mime-types``를 통해 커스터마이징할 수 있다.

<img width="967" alt="스크린샷 2021-09-12 오전 12 13 25" src="https://user-images.githubusercontent.com/56240505/132952624-6147e96e-f326-4fae-bdb4-e34e945294c8.png">

* GZIP 적용 전 index.js의 크기는 3.3KB다.

<img width="967" alt="스크린샷 2021-09-12 오전 12 15 12" src="https://user-images.githubusercontent.com/56240505/132952744-f9254f5e-6e04-4f08-87a6-9078fda09ab3.png">

* GZIP 적용 이후 index.js의 크기는 1.4KB다.

### 3.3. 압축 관련 HTTP 메시지

![image](https://user-images.githubusercontent.com/56240505/132952815-072ea8b2-740d-4c20-a832-488580af71dd.png)

* 브라우저는 브라우저가 지원하는 압축 알고리즘 및 우선순위를 Accept-Encoding 헤더에 담아 전송한다.

![image](https://user-images.githubusercontent.com/56240505/132952732-62988fb1-14f7-4ced-8e55-f42c2ccd9296.png)

* 서버는 해당 헤더를 뽑아내서 응답의 바디를 압축하는데 사용하고, 서버가 선택한 알고리즘을 Content-Encoding 헤더를 사용해 브라우저에게 알려준다.
* 서버가 압축하면 중간 노드(프록시 서버 등)은 압축하지 않다가, 종단 클라이언트(브라우저)에 이르러서야 압축을 해제한다.

<br>

## 4. HTTP 캐싱

네트워크에서의 캐싱이란 특정 요청의 응답 리소스의 복사본을 재사용하는 것을 의미한다.

* 응답받은 정적 컨텐츠를 로컬 브라우저 혹은 CDN 서버 등에 저장해두고, 동일한 요청이 발생하면 해당 위치에서 불러온다.
  * 매번 동일한 요청에 대해 웹 서버가 처리 및 응답하지 않게 되면서, 응답 시간이 줄어들고 웹 서버로 향하는 트래픽이 감소된다.
* 일반적으로 GET 응답만 캐싱한다.
* 캐싱 키는 GET 요청과 URI다.

### 4.1. 기본 헤더

* Cache-Control
  * Cache-Control:public
    * 클라이언트 브라우저 이외의 중간 프록시 등에도 응답 캐싱을 허용한다.
  * Cache-Control:private
    * 클라이언트 브라우저만 캐싱을 허용하며, 중간 프록시 및 CDN은 불허한다.
    * 기본값이다.
  * Cache-Control:public, max-age=31536000
    * max-age는 초 단위로 캐시하는 시간 범위를 설정한다.
    * 31536000은 1년이며, 0으로 설정하면 캐시하지 않는 것과 동일하다.
* Expires
  * 지정한 만료 날짜까지만 캐시를 허용한다.
  * 이후부터는 브라우저가 새로운 복사본을 요청한다.
  * 만료 날짜 이전에 수정이 필요한 경우 문제가 발생한다.

### 4.2. 조건부 요청

![image](https://user-images.githubusercontent.com/56240505/132953717-a4dd9c91-dfb9-49b1-ba81-6a2b29526787.png)

브라우저는 캐시된 리소스를 사용하기 전에, 서버에 해당 리소스가 업데이트되었는지 확인하는 조건부 요청을 보낸다.

* 업데이트된 리소스가 없으면 서버는 HTTP Status 304와 빈 응답 본문을 생성하며, 브라우저는 캐시된 자원을 이용한다.
* 변경되었다면 서버는 새로운 버전의 리소스와 함께 OK 200 응답을 보낸다.

```
Cache-Control:public, max-age=31536000
Last-Modified: Mon, 03 Jan 2011 17:45:57 GMT
```

* 캐싱되는 최초 자원의 응답 헤더에 Last-Modified가 있으면 조건이 활성화된다.
* 브라우저는 날짜를 저장해두고 다음에 해당 리소스를 요청 할 때 헤더에 If-Modified-Since를 추가해 조건부 요청을 보낸다.

```
If-Modified-Since: Mon, 03 Jan 2011 17:45:57 GMT
```

* 리소스 날짜가 동일하면 서버는 304를 반환한다.
* 이러한 방식을 Time Based라고 한다.

```
Cache-Control:public, max-age=31536000
ETag: "15f0fff99ed5aae4edffdd6496d7131f"
```

* ETag 방식은 캐싱되는 최초 자원의 응답 헤더에 ETag가 있으면 조건이 활성화된다.
* Last-Modified로 마지막 수정 날짜 확인이 어려울 때 사용한다.
* 브라우저는 조건부 요청을 보낼 때, ETag 값을 다음과 같이 요청 헤더에 붙인다.

```
If-None-Match: "15f0fff99ed5aae4edffdd6496d7131f"
```

* 마찬가지로 ETag 값이 동일하면 서버는 304를 반환한다.

### 4.3. Spring Boot 적용

> application.yml

```yml
spring:
  web:
    resources:
      cache:
        cachecontrol:
          cache-private: true
          max-age: 3600
```

* 혹은 Configuration 클래스에서 ResourceHandler에 캐싱 설정을 수행할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/132954883-e6b42e4c-1d83-4d33-901d-179c325a815e.png)

* 정적 리소스 파일의 응답에 대해 Cache-Control이 적용된다.
* 이후 요청은 디스크 캐시를 이용하거나 if-modified-since 등의 조건부 요청이 나간다.

> WebConfig.java

```java
@Bean
public FilterRegistrationBean<ShallowEtagHeaderFilter> filterFilterRegistrationBean() {
    FilterRegistrationBean<ShallowEtagHeaderFilter> filterFilterRegistrationBean =
        new FilterRegistrationBean<>(new ShallowEtagHeaderFilter());
    filterFilterRegistrationBean.addUrlPatterns("/api/**");
    filterFilterRegistrationBean.setName("etagFilter");
    return filterFilterRegistrationBean;
}
```

* ETag 관련 필터를 Bean으로 등록하고, ETag를 사용할 API URL을 맵핑해준다.
* 테스트를 위해 모든 컨트롤러에 ETag를 설정했다.

> PageController.java

```java
@GetMapping(value = "/{id}/custom-etag")
public ResponseEntity<Foo>
  findByIdWithCustomEtag(@PathVariable("id") final Long id) {

    // ...Foo foo = ...

    return ResponseEntity.ok()
      .eTag(Long.toString(foo.getVersion()))
      .body(foo);
}
```

* 이와 같이 ResponseEntity에 ETag 생성 로직을 부여할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/132955101-ed835d30-7d17-4868-9672-98774c27992e.png)

* 이후 해당 API의 응답 헤더에 ETag가 부착되며, 동일 API 요청이 발생하는 경우 if-none-match 조건부 요청을 통해 304가 반환됨을 알 수 있다.

<br>

---

## References

* 우아한테크코스
