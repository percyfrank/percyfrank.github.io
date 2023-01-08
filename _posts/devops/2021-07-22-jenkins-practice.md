---
title: "Jenkins Freestyle CI/CD 톺아보기"
excerpt: "손너잘이 주관하는 CI/CD 특강!"
categories:
  - DevOps
tags:
  - DevOps
date: 2021-07-22
last_modified_at: 2021-07-22
---

## 1. Jenkins 설치

> Shell

```bash
sudo apt update && sudo apt install -y docker.io
sudo usermod -aG docker ubuntu
```

* AWS EC2(Ubuntu) 인스턴스에 Jenkins를 설치한다.
* 해당 인스턴스 로컬 환경에 바로 설치할 수 있으나 Docker를 이용해본다.
* Docker 그룹에 Ubuntu를 추가한 뒤 쉘을 재접속한다.

> Shell

```bash
cat /etc/group | grep docker
docker:x:115:ubuntu
```

* 위 처럼 나오면 성공적으로 그룹에 추가된 것이다.

> Shell

```bash
docker run -p 8080:8080 -d --name jenkins jenkins/jenkins:jdk11
```

* JDK 8 버전을 사용하고 싶다면 ``:lts``로 옵션을 변경한다.
* 볼륨 마운팅을 하고 싶다면 ``-v ${path}:${path}`` 옵션을 추가한다.

<br>

## 2. Jenkins 로그인

![image](https://user-images.githubusercontent.com/56240505/126646664-3d46930f-3d94-42f9-ab8f-aed809f1f23a.png)

* Jenkins EC2 Public IP에 접속하면 자동으로 위와 같은 로그인 화면이 로드된다.

> Shell

```bash
docker logs jenkins
```

![1](https://user-images.githubusercontent.com/56240505/126648390-31e1ba0c-2dce-4b02-8f52-25cc02cdd7d2.png)

* 쉘에 ``docker logs ${jenkins 컨테이너 ID 혹은 Name}``을 입력하면 다음과 같은 로그가 출력된다.
* 그 중, 검게 칠한 해시된 패스워드를 Jenkins 로그인 페이지에 입력한다.

![image](https://user-images.githubusercontent.com/56240505/126648542-547fcd87-09ec-4eca-8a72-cee5f8eedafb.png)

* Jenkins가 제안하는 옵션 구성들로 설치하면 된다.

<br>

## 3. Github CI

[Github 개발자 설정](https://github.com/settings/tokens) 탭에서 Personal Access Token을 받는다. Scope(허용 권한)는 admin:repo_hook와 repo를 지정해준다.

![image](https://user-images.githubusercontent.com/56240505/126649251-7ff09ef2-7892-4f07-9f9c-b18b5a2324b3.png)

* 빌드를 하는데 있어서 Repo, IP, Access Token 등 민감한 정보를 관리하는 창으로 이동한다.

![image](https://user-images.githubusercontent.com/56240505/126649371-40eb9b5a-2370-4775-9c7c-6b1de70bef07.png)

* Username은 Jenkins를 사용하고자 하는 Github 계정명을, Password는 Access Token을 입력한다.
* ID 및 Description은 본인이 쉽게 식별할 수 있도록 입력한다.

![image](https://user-images.githubusercontent.com/56240505/126649619-a7d9b9a6-180e-44a8-a2d5-37b4ff528a64.png)

* 참고로 Jenkins 시스템설정(환경설정) 탭 하단에서 이렇게 얻어온 Access Token이 유효한지, 서버와의 Connection을 테스트할 수 있다.
* 이후 Jenkins 메인 페이지에서 ``새로운 Item - Freestyle Project``를 선택하여 CI를 적용할 아이템을 생성한다.
  * Github Project를 체크한다.

![image](https://user-images.githubusercontent.com/56240505/126650288-509c6496-d831-4a08-abee-dab21498cee0.png)

* 소스 코드 관리를 위와 같이 Git으로 지정하고, CI를 적용할 Repository URL과 해당 Github 계정에 대한 Access Token을 지정한다.
* 빌드할 타깃 Branch 이름 또한 지정한다.

![image](https://user-images.githubusercontent.com/56240505/126650574-4c08f008-2861-4d6c-9eb2-3d879cb8eead.png)

* Jenkins를 통해 CI를 관리할 때, 어떤 방법으로 빌드를 유발할지 선택한다.
* 이번 글에서는 타깃 Repository에 어떠한 이벤트가 발생했을 때 빌드하는 Hook Trigger를 사용한다.

![image](https://user-images.githubusercontent.com/56240505/126650865-8c42fae3-3728-486d-a812-7975994359cb.png)
![image](https://user-images.githubusercontent.com/56240505/126650909-ef37b5c9-5d46-45e6-90c8-9d3a2aea9d8b.png)

* Build 방식은 Shell Script를 선택한다.
* 커맨드가 동작하는 위치는 Repository의 루트 위치이며, 이를 참고하여 프로젝트를 빌드하는 스크립트를 작성한다.

![image](https://user-images.githubusercontent.com/56240505/126651343-3460c784-0c65-49d0-8711-878edf596785.png)

* 또한 빌드 후 Email 알림 등 추가 조치를 선택할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/126651584-afcf1aa5-e188-480d-b603-c6f57b03db3d.png)

만약 Submodule을 사용 중이라면 Additional Behaviours에 Submoudle 관련 설정을 추가해야 한다.

* 위와 같이 설정해주고, Path는 CI 대상이 되는 Repository 내부에 존재하는 Submodule 디렉토리의 경로를 입력한다.
  * 예) ``backend/pick-git/security``

Jenkins 메인 페이지에서 Build Now를 실행시키고 성공하면, Docker 컨테이너 내부 ``/var/jenkins_home/workspace/`` 경로에 CI 대상으로 지정해둔 Repository 폴더가 존재한다.

* 해당 Repository 폴더 내부에서 lib 디렉토리를 잘 살펴보면 JAR 파일이 존재한다.
* 즉, 제대로 Build Script가 동작했음을 알 수 있다.

### 3.1. Github Webhook 추가

![image](https://user-images.githubusercontent.com/56240505/126652664-20d3be0e-26d5-4a33-bf70-c17a31c0968a.png)
![image](https://user-images.githubusercontent.com/56240505/126652689-8244b36c-65bc-4791-80c4-a763de030e77.png)

Jenkins에서 Build Now를 눌러 수동 빌드하는 것이 아니라, 타깃 Github Repository에 특정 이벤트가 발생하면 자동으로 Jenkins가 빌드 등 CI를 진행하도록 사진처럼 Webhook을 추가한다. Payload URL은 내 Jenkins 서버 IP와 포트를 사진의 형식대로 작성한다.

* **주의 : http://{IP}:{Port}/github-webhook/** 이라 작성해야 정상적으로 동작한다.
  * 마지막 Slash를 빼먹으면 동작하지 않는다.
* Webhook을 유발하는 방식으로는 Push일 때 혹은 모든 이벤트일 때 등을 지정할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/126653213-d505c8c1-b84a-4305-8709-ea239b371fc7.png)

Pull Request가 타깃 브랜치에 Merge되거나, 혹은 타깃 브랜치에 Push하면 해당 정보들이 모두 Jenkins 서버로 전송된다.

* Jenkins는 해당 정보를 파싱하여 자동으로 빌드 등 CI 작업을 진행한다.
* 실제로는 PR에 달린 라벨 정보 등을 바탕으로 Jenkins가 CI/CD 작업을 선별적으로 수행하는 등 더 복잡한 파이프라인을 구성할 수 있다!

<br>

## 4. CD

빌드 후 조치로 [Jenkins가 빌드된 JAR 결과물을 원격(Remote) 서버로 배포](https://yookeun.github.io/tools/2018/04/14/jenkins-remote/)해야 한다. 다양한 방법이 있지만 Jenkins에서 **Publish Over SSH**라는 플러그인을 깔아 진행한다.

![2](https://user-images.githubusercontent.com/56240505/126654451-6322f3ed-c660-479e-92ee-88ad6aa683ec.png)

* 플러그인이 정상 설치되면, Jenkins 시스템설정(환경설정) 탭 하단에 Publish Over SSH라는 탭이 활성화된다.
* 해당 탭에 배포할 서버의 RSA Key와 Hostname(배포 서버 Private IP) 및 Username(배포 서버의 ubuntu 등 Username)을 입력한다.

![image](https://user-images.githubusercontent.com/56240505/126654999-56534568-ee1c-42bf-b93b-d224d989e22d.png)

* 배포 중인 Jenkins 프로젝트 아이템 구성에서 빌드 환경을 위와 같이 세팅해준다.
  * 배포할 SSH Server를 선택한다.
  * Jenkins 서버에 위치한 JAR 빌드 결과물(Source Files)를 배포 서버로 자동 전송된다.
  * 전송할 때 앞의 장황한 디렉토리 접두사를 자동으로 삭제해서 보내준다.

![image](https://user-images.githubusercontent.com/56240505/126655576-b7bf02b1-d8cd-44eb-a448-a6f47c813346.png)

* 배포 서버로 빌드 결과물이 도착하면 수행할 명령문을 입력한다.
  * ``nohup java -jar *.jar -Dspring.profiles.active=prod`` 등.
  * 프로젝트의 빌드(CI)가 완료되면 자동으로 배포 서버에서 해당 프로젝트가 배포되도록 명령어를 짤 수 있다.

<br>

## 5. 마치며

Jenkins를 활용한 CI/CD Flow는 다음과 같다. 현업에서는 Freestyle보다는 복잡한 Pipeline을 사용한다.

1. 타깃 Repository의 Branch에 Push 등 이벤트가 발생하면 해당 정보가 명시된 Jenkins 서버로 이동한다.
2. Jenkins 서버는 해당 정보를 바탕으로 타깃 Repository 프로젝트를 주어진 스크립트를 바탕으로 빌드한다.
3. 빌드가 완료되면 Jenkins 서버는 자신의 서버에 위치한 빌드 결과물(JAR 등)을 배포 서버로 전송한다.
4. 배포 서버는 빌드 결과물을 전달 받으면, 마찬가지로 주어진 스크립트를 바탕으로 해당 빌드 결과물을 실행하여 배포한다.

<br>

---

## References

* [손너잘](https://github.com/bperhaps)
