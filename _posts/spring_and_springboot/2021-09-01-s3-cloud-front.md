---
title: "Spring Boot AWS S3 이미지 업로드"
excerpt: "AWS S3 및 CloudFront를 응용하여 이미지 파일을 관리하자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-09-01
last_modified_at: 2021-09-01
---

## 1. 용어 정리

* AWS S3(Simple Storage Service) : 파일 서버의 역할을 하는 서비스다.
* AWS CloudFront : AWS에서 제공하는 CDN 서비스로서, 정적 파일 등을 캐싱을 통해 사용자에게 파일을 빠른 속도로 전송 및 제공하는 것을 목표로 한다.
* CDN(Content Delivery Network) : 지리적으로 분산된 여러 개의 서버다.
  * 웹 콘텐츠를 사용자와 가까운 곳에서 전송함으로써 전송 속도를 높인다.

### 1.1. CloudFront + S3를 정적 서버로 활용

팀 프로젝트를 진행하면서 정적인 파일을 별도의 EC2 웹 서버에 배포하는 과정을 생략했다.

* 웹 서버는 HTTP Method를 통해서 CRUD의 기능을 제공하지만, 웹 서버로부터 필요한 기능은 R이다.
  * S3(Cloud Object Storage, 정적 파일 객체 스토리지)를 통해 R 기능을 만족시킬 수 있다.
  * 굳이 번거로운 설정을 요구하는 웹 서버를 띄울 필요가 없다.
* 웹 서버에서도 캐싱 기능을 직접 설정해서 사용할 수 있긴 하지만, AWS CloudFront(CDN)을 이용해서 AWS에서 제공하는 캐싱 기능의 이점을 가지고 갈 수 있다.
* 따라서, 웹 서버를 별도로 띄우지 않고 S3를 프론트엔드용 정적 서버로 활용한다.

<br>

## 2. S3 버켓 생성

![image](https://user-images.githubusercontent.com/56240505/131600869-7d773c22-ab45-4261-9f9a-caabeec6494b.png)

* 현재 우테코 AWS 계정은 S3 Public Access 오픈이 불가능하다.
* 아울러 S3에 대한 AccessKey 및 SecretKey 발급도 불가능한 상태다.
* 따라서 읽기 작업은 CloudFront를 통해 접근하도록 구성하며, 쓰기 작업은 S3 쓰기 권한이 부여된 IAM 역할이 부여된 인스턴스에서만 가능하다.

<br>

## 3. CloudFront 설정

![image](https://user-images.githubusercontent.com/56240505/131601817-9efc4806-cbec-46c8-bb97-e146d11377e7.png)

* ``CloudFront - 배포 생성`` 을 선택한다.
* 원본 도메인은 이전에 생성한 S3 버켓을 선택한다.
* S3 버킷 액세스에 대해 OAI 사용을 선택하고 ``새 OAI 생성``을 눌러 OAI ID를 넣어준다.
* Origin Shield는 캐시 적중률을 높여 오리진의 부하를 줄이는 데 도움이 되는 중앙 집중식 캐싱 계층이다.
  * 리퀘스트의 횟수를 줄여 관리비용을 줄인다.

### 3.1. S3 버켓 정책 수정

S3 버켓의 ``권한 탭 - 버켓 정책``에 다음 내용을 기입한다.

```
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity ${OAI ID}"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::[S3 버켓 이름]/*"
        }
    ]
}
```

<br>

## 4. Spring Boot 프로젝트 설정

> build.gradle

```groovy
implementation platform('com.amazonaws:aws-java-sdk-bom:1.11.1000')
implementation 'com.amazonaws:aws-java-sdk-s3'
implementation group: 'org.apache.tika', name: 'tika-core', version: '1.26'
```

* 개인적으로 쓰기 작업에 대한 권한(AccessKey 및 SecretKey)를 부여 받을 수 있다면, 해당 정보를 yml에 저장해두고 로컬에서도 업로드가 가능하다.
* 현재는 IAM 역할이 부여된 인스턴스에서만 쓰기 작업이 가능하기 때문에 이를 생략한다.
  * 따라서 로컬에서 이미지 업로드 테스트를 하기 어려우며, LocalStack을 사용해야 한다.

> S3Configuration.java

```java
@Configuration
public class S3Configuration {

    @Bean
    public AmazonS3 amazonS3() {
        return AmazonS3ClientBuilder
            .standard()
            .withCredentials(InstanceProfileCredentialsProvider.getInstance())
            .withRegion(Regions.AP_NORTHEAST_2)
            .build();
    }
}
```

* IAM 역할이 부여된 AWS EC2 인스턴스의 자격 증명을 사용한다.

<br>

## 5. S3 업로드 예제

> S3Storage.java

```java
@Repository
public class S3Storage implements FileStorage {

    @Value("${aws.s3.bucket_name}")
    private String bucket;

    @Value("${aws.cloud_front.file_url_format}")
    private String fileUrlFormat;

    private final AmazonS3 amazonS3;

    private final FileNameGenerator fileNameGenerator;

    public S3Storage(AmazonS3 amazonS3, FileNameGenerator fileNameGenerator) {
        this.amazonS3 = amazonS3;
        this.fileNameGenerator = fileNameGenerator;
    }

    @Override
    public List<StoreResult> store(List<MultipartFile> files, String userName) {
        return files.stream()
            .map(file -> upload(file, userName))
            .collect(Collectors.toList());
    }

    private StoreResult upload(MultipartFile file, String userName) {
        String fileName = fileNameGenerator.generateFileName(file, userName);
        ObjectMetadata objectMetadata = generateObjectMetaData(file);
        try {
            amazonS3.putObject(bucket, fileName, file.getInputStream(), objectMetadata);
            return new StoreResult(fileName, String.format(fileUrlFormat, fileName));
        } catch (Exception e) {
            throw new UploadFailureException();
        }
    }

    private ObjectMetadata generateObjectMetaData(MultipartFile file) {
        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentLength(file.getSize());
        objectMetadata.setContentType(file.getContentType());
        return objectMetadata;
    }
}
```

* Controller에서 받은 MultipartFile 형식의 파일을 AmazonS3 인스턴스의 ``putObject()`` 메서드를 통해 S3로 업로드한다.
* 업로드할 이미지 파일 이름의 경우 중복되지 않도록 적절히 처리해서 생성해준다.
* 업로드가 성공하면 해당 이미지 파일을 참조할 수 있는 URL 링크를 생성해서 반환한다.

> application.yml

```yml
aws:
  cloud_front:
    file_url_format: https://d2lkb1z978lt7d.cloudfront.net/images/%s
  s3:
    bucket_name: xlffm3-test/images
```

* 현재 ``xlffm3-test`` 라는 버킷의 ``/images``라는 디렉토리에 이미지 파일이 쌓이도록 구성해두었다.

![image](https://user-images.githubusercontent.com/56240505/131605379-d0b13d1b-441f-4a80-adf7-52c30695e47c.png)

<img width="538" alt="스크린샷 2021-09-01 오후 12 09 43" src="https://user-images.githubusercontent.com/56240505/131605520-039941d1-f2c0-4139-9609-b9821b085b24.png">

* 이미지 업로드 테스트 결과, ``fileNameGenerator.generateFileName(file, userName);``를 통해 생성한 임의의 이름의 이미지 파일이 잘 저장되는 것을 확인했다.

![image](https://user-images.githubusercontent.com/56240505/131605860-402442d8-2ecb-462e-ac53-1b52e4e87167.png)

* 본인이 배포한 S3 도메인으로 접근할 수 있는 CloudFront URL이 바로 ``도메인 이름``이다.
* 따라서 업로드에 성공한 이미지 파일을 참조할 수 있는 URL은 ``https://{도메인 이름}/{파일 이름}``이 된다.
  * 문자열을 적절히 파싱해서 업로드된 이미지의 URL을 생성해 반환한다.

### 5.1. 파일 이름 UUID 지정

업로드된 이미지 파일의 이름이 중복되지 않도록 한다. 고유한 이름을 가질 수 있게 UUID로 파일명을 지정 후, 업로드를 진행한다.

* v1: 타임스탬프(시간) 기준으로 생성
* v3: MD5 해시 기준으로 생성
* v4: 랜덤값을 기반으로 생성
* v5: SHA-1 해시 기준으로 생성

> FileNameGenerator.java

```java
@Component
public class FileNameGenerator {

    private static final Tika TIKA = new Tika();

    private final HashClock hashClock;

    public FileNameGenerator(HashClock hashClock) {
        this.hashClock = hashClock;
    }

    public String generateFileName(MultipartFile multipartFile, String userName) {
        String hashFileName = applyMD5(multipartFile, userName);
        String extension = detectFileExtension(multipartFile);
        return hashFileName + extension;
    }

    private String applyMD5(MultipartFile multipartFile, String userName) {
        String fileName = multipartFile.getOriginalFilename();
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] bytes = (fileName + userName + hashClock.now().getNano())
                .getBytes(StandardCharsets.UTF_8);
            md.update(bytes);
            return Hex.encodeHexString(md.digest());
        } catch (NoSuchAlgorithmException e) {
            throw new HashFailureException();
        }
    }

    private String detectFileExtension(MultipartFile multipartFile) {
        try {
            MimeTypes defaultMimeTypes = MimeTypes.getDefaultMimeTypes();
            String target = TIKA.detect(multipartFile.getBytes());
            MimeType mimeType = defaultMimeTypes.forName(target);
            return mimeType.getExtension();
        } catch (IOException | MimeTypeException e) {
            throw new FileExtensionException();
        }
    }
}
```

업로드한 MultipartFile 이름과 업로드 유저 이름 및 시간을 조합시키고 MD5 알고리즘을 적용해 새로운 파일 이름을 만들어낸다.

* 뒤이어 Tika 라이브러리를 통해 주어진 MultipartFile의 MimeType을 알아내어 주어진 파일의 확장자를 확인한다.
  * ``generateFileName()`` 메서드는 새롭게 생성한 파일 이름과 확장자를 붙여서 반환한다.
  * 예) ``스크린샷20210303.png`` -> ``0adfaf123123132.png``

MIME(Multipurpose Internet Mail Extensions)은 다음과 같다.

* 클라이언트에게 전송된 문서의 다양성을 알려주기 위한 메커니즘이다.
  * 웹에서 파일의 확장자는 별 의미가 없다.
* 파일을 인터넷으로 전송할 때, 바이너리 데이터로 인코딩된다.
* MIME은 인코딩된 바이너리 데이터(사진, 동영상 등)가 어떤 타입인지 명시해준다.

<br>

---

## References

* [CloudFront를 통해 S3 액세스 하기](https://hyeon9mak.github.io/access-s3-through-cloudfront/)
* [[스프링] AWS S3에 이미지 업로드 하기](https://leveloper.tistory.com/46)
* [AWS 기반 Spring Boot 애플리케이션 개발 시작하기](https://aws.amazon.com/ko/blogs/korea/getting-started-with-spring-boot-on-aws/)
* [SpringBoot & AWS S3 연동하기](https://jojoldu.tistory.com/300)
* [AWS SAA S3 병목현상을 막기 위한 버켓 파티셔닝 -  Hex Hash](https://m.blog.naver.com/naebon/221756312403)
* [AWS S3에 파일 업로드](https://phsun102.tistory.com/14)
