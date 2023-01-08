---
title: "Git Submoudle을 통한 민감 정보 관리"
excerpt: "Access & Secret Key 정보를 별도로 관리해보자."
categories:
  - Git
tags:
  - Git
date: 2021-08-17
last_modified_at: 2021-08-17
---

## 1. Submoudle

* Git 저장소 안에 다른 Git 저장소를 디렉토리로 분리해 넣는 것이 서브모듈이다.
* 다른 독립된 Git 저장소를 Clone해서 내 Git 저장소 안에 포함할 수 있으며, 각 저장소의 커밋은 독립적으로 관리한다.

<br>

## 2. 명령어 모음

> Shell

```bash
$ git submodule add ${추가할 private 서브 모듈 저장소}
$ git submodule update --remote --merge #프로젝트 내 서브모듈 최신화
$ git clone --recurse-submodules ${서브 모듈을 포함하는 프로젝트 저장소} #소스 코드를 내려 받을 때 서브 모듈을 함께 받음
```

* Spring Cloud Config 혹은 AWS Service Manager 등의 외부 서비스를 활용하는 방법도 고려해보자.

### 2.1. Gradle 설정

> build.gradle

```groovy
processResources.dependsOn('copySecurity')

task copySecurity(type: Copy) {
    from './submodule/application-security.yml'
    into './src/main/resources'
}
```

* 사용할 서브 모듈 데이터를 클래스 패스로 복사한다.
  * 해당 데이터는 반드시 .gitignore에 추가한다.

<br>

---

## References

* [7.11 Git 도구 - 서브모듈](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88)
