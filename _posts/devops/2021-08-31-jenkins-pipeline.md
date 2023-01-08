---
title: "Jenkins Pipeline CI/CD 구현"
excerpt: "Webhook 이벤트를 통해 서비스를 자동 배포한다."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-08-31
last_modified_at: 2021-08-31
---

## 1. Why Not Travis?

우리 팀이 느낀 Travis의 단점은 다음과 같다.

* 상용 서비스용 유료 플랜이 별도로 존재한다.
* 플러그인이 Jenkins에 비해 다양하지 않고, 레퍼런스 또한 적다.
* Travis-CI는 로컬의 CI 환경과 동일한 빌드 환경을 제공하지 않는다.
* ``.travis.yml`` 파일을 수정하고 테스트하려면 git push를 반복해야 한다.

<br>

## 2. EC2 Docker 설치 및 Jenkins 컨테이너 실행

> Shell

```bash
$ sudo apt update && sudo apt install -y docker.io && sudo apt install -y docker-compose
$ sudo usermod -aG docker ubuntu
```

* 이후 ``docker-compose.yml`` 파일을 작성한다.
  * 개별적인 명령어를 통해 이미지를 받고 실행할 수 있으나, 파일로 관리하는 것이 편하기 때문이다.

> docker-compose.yml

```bash
version: '3'
services:
   jenkins:
      image: jenkins/jenkins:lts #JDK11을 사용하려면 lts가 아닌 jdk11 명시
      container_name: jenkins-conatiner
      ports:
        - 8080:8080
      volumes:
        - /home/jenkins:/var/jenkins_home
```

* 설정은 취향에 따라 커스터마이징한다.

> Shell

```bash
$ sudo chown 1000 /home/jenkins
$ docker-compose up -d
```

* 볼륨 마운팅할 디렉토리를 ``/home/jenkins``로 두었는데, 해당 디렉토리에 적절한 권한을 부여해야 컨테이너가 정상 실행된다.
* 권한 부여 후 컨테이너를 실행한다.

<br>

## 3. Jenkins 설정

``http://{jenkins-ec2-public-ip}:{port}``로 접속해서 Jenkins를 설정한다.

> Shell

```bash
$ cat {볼륨_마운팅_디렉토리}/secrets/initialAdminPassword
```

* 초기 비밀키를 요구하면 명령어 실행 결과를 붙여넣는다.
* 이후 Suggested Plugins를 선택하고 어드민 계정 아이디 및 비밀번호를 설정한다.

<br>

## 4. Pipleline 프로젝트 시작

``Jenkins 관리 - 플러그인 관리`` 탭에서 Generic Webhook Trigger 플러그인을 먼저 설치한다. 이후 ``새로운 Item - Pipeline``을 선택해 본격적으로 Pipeline을 구축한다. 단순히 특정 Branch에 코드가 Push됬을 때 빌드 및 배포할 계획이라면 [Freestyle Project](https://xlffm3.github.io/devops/jenkins-practice/) 또한 충분하다.

이번 글에서는 Webhook을 활용해 조금 더 세밀한 작업을 진행하고자 Pipeline을 사용한다.

* Freestyle과는 다르게 Generic Webhook Trigger는 특정 브랜치에 코드 Push가 발생한 것을 감지하는 기능을 **기본적으로 제공하지 않는다.**
* Payload 등을 적합하게 파싱함으로써 빌드를 유발시킬지 결정한다.

### 4.1. Pipeline 문법

1. Declarative Pipeline - Pipeline
  * 보다 쉽게 작성 할 수 있게 커스텀 되어 있다.
  * Groovy-syntax 기반이며, Groovy 문법을 잘 몰라도 작성이 가능하다.
2. Scripted Pipeline - Node
  * Groovy 기반이며, 선언형보다 효과적으로 많은 기능을 포함하여 작성할 수 있다.
  * Groovy 문법에 대한 러닝 커브 등 작성 난이도가 높다.

<br>

## 5. Pipeline Script 작성 : Build & Test

### 5.1. Webhook Build Trigger 설정 및 이해

<img width="586" alt="스크린샷 2021-08-30 오후 11 18 39" src="https://user-images.githubusercontent.com/56240505/131517498-1b2b1151-2cd3-4f1d-94f4-9d4c549e4743.png">

* GitHub Project를 선택하고 프로젝트 URL을 입력한다.

<img width="646" alt="스크린샷 2021-08-30 오후 11 23 19" src="https://user-images.githubusercontent.com/56240505/131517658-540aef61-a3b2-4ebb-8622-fcc480c80d06.png">

* Build Triggers 탭에서 Generic Webhook Trigger를 체크한다.

![image](https://user-images.githubusercontent.com/56240505/131517805-bbed551c-34b5-4e24-80ee-de8a30fccc75.png)

* Token 부분은 원하는 text를 입력한다.
  * 필자는 심플하게 token이라고 붙였다.
* 해당 토큰이 붙은 Webhook 요청에 한해서 Jenkins Pipeline이 유발된다.

<img width="840" alt="스크린샷 2021-08-30 오후 11 26 18" src="https://user-images.githubusercontent.com/56240505/131518158-ff79a51a-cb06-46e9-8564-db6a1973dd6e.png">

CI/CD 대상이 되는 GitHub 프로젝트 레포지토리에서 ``Settings - Webhooks`` 탭으로 들어가 새로운 Webhook을 추가한다.

* Payload URL은 Generic Webhook Trigger가 제시하는 ``http://{jenkins-ec2-public-ip}:{port}/generic-webhook-trigger/invoke``를 따른다.
  * 해당 URL의 Suffix에 예전에 기입해둔 Token 정보를 ``?token=token``이라는 쿼리 스트링으로 붙인다.
* Content-Type은 ``application-json``으로 설정한다.
* 세밀한 Pipleline 구축을 위해 ``Send me everything`` 옵션을 선택한다.
  * PR 및 Issue 등의 Open과 Close 등 다양한 Webhook을 Jenkins로 전달하게 된다.
  * PR의 Label을 감지하는 등 세밀한 Pipeline 구축이 필요 없다면 ``Just the push event``도 충분하다.

<img width="720" alt="스크린샷 2021-08-30 오후 11 26 42" src="https://user-images.githubusercontent.com/56240505/131519706-4b3ec957-b54c-4e7d-bc6b-bd18203d94c0.png">

* 우측 ``Recent Deliveries`` 탭으로 가면 GitHub이 발생한 Webhook Event를 Jenkins에 전송한 이력이 남아있다.
  * 코드 푸시 혹은 PR 생성 등을 통해 두 서버간 통신이 원활하게 이루어지는지 확인해본다.
  * 혹은 ``Redeliver`` 버튼을 눌러 이전에 보낸 요청을 다시 보냄으로써 테스트를 진행할 수 있다.

<img width="270" alt="스크린샷 2021-08-30 오후 11 27 27" src="https://user-images.githubusercontent.com/56240505/131520137-3a80d731-b8ee-4f5d-b373-f383d3a0ee3a.png">

* GitHub이 보낸 요청의 Payload를 보면 다양한 정보가 전송됨을 확인할 수 있다.
* Payload에서 원하는 데이터를 추출하고, 조건에 부합하는 경우 빌드를 유발시킨다.

<img width="828" alt="스크린샷 2021-08-30 오후 11 28 50" src="https://user-images.githubusercontent.com/56240505/131520403-c7a69657-bf29-4ef9-8e56-9e2880a39817.png">

* ``Post content parameters`` 추가 버튼을 클릭한다.

<img width="565" alt="스크린샷 2021-08-30 오후 11 29 50" src="https://user-images.githubusercontent.com/56240505/131520522-75d9779a-2412-4ee8-abb7-bab3b4053aab.png">

Variable은 추출한 Payload 값의 변수명을 기입하고, Expression은 Webhook을 통해 전달받은 Payload에서 원하는 값에 대한 JSON Path 표현식을 작성한다. JsonPath 표현식은 Reference를 참고하자.

위 사진은 해당 PR의 Merge 여부에 관한 값이다. Payload JSON을 참고하여 필요한 Variable을 추가해준다.

* LABELS : ``$.pull_request.labels..name``
  * 복수 개의 라벨이 올 수 있기 때문이다.
* BRANCH : ``$.pull_request.base.ref``

<img width="385" alt="스크린샷 2021-08-31 오전 12 20 36" src="https://user-images.githubusercontent.com/56240505/131521942-21324d56-ba04-4d00-8fb0-49c1343c8316.png">
<img width="420" alt="스크린샷 2021-08-31 오전 12 20 11" src="https://user-images.githubusercontent.com/56240505/131521953-2932a402-ca43-43e3-9a06-8b4800a08005.png">

Payload 데이터의 조건에 따라 Pipeline 스크립트 실행할지 여부를 결정하는 필터를 설정한다. Payload로 전달받아 값을 할당한 Text에 선언된 변수들의 값이 Expression에 기입된 값 혹은 표현식과 일치하면 스크립트를 실행한다.

* 사진의 경우, ``backend`` 라벨을 단 PR이 master(좌) 혹은 develop(우) 브랜치로 Merge 되었을 때 Pipeline 스크립트를 실행한다.
* 이를 응용하면, 내가 원하는 Label이나 작성자가 생성한 PR이 특정 브랜치로 머지될 때 유발되는 Pipeline을 작성할 수 있다.

<img width="323" alt="스크린샷 2021-08-30 오후 11 54 54" src="https://user-images.githubusercontent.com/56240505/131521431-703953fd-1f1c-45eb-9d1e-71adb4b931a3.png">

* 간단하게 테스트용 Pipeline 스크립트를 작성한다.
  * ``stages - stage - steps``로 이루어진 절차 및 관련 문법은 Reference를 참고하자.

<img width="387" alt="스크린샷 2021-08-30 오후 11 55 11" src="https://user-images.githubusercontent.com/56240505/131522985-b86c3c2f-ef61-4d54-a99c-4080c0a7d8d3.png">

* 기대하는 조건으로 PR 생성 및 Merge를 진행하면 Pipeline이 실행되고 Payload 값을 ``echo`` 출력하는 것을 볼 수 있다.

### 5.2. Checkout 및 Submodule 설정

GitHub Repository 프로젝트의 특정 브랜치를 빌드 및 테스트하기 위해서는 어떤 작업이 수반되어야 할까?

* 사람이라면 해당 프로젝트 코드를 Clone받고 특정 브랜치로 Checkout한 뒤, 빌드 및 테스트를 수행할 것이다.
* Jenkins 또한 해당 프로젝트 코드를 Clone받고 특정 브랜치로 Checkout한 뒤, 개발자가 작성한 스크립트(빌드 및 테스트)를 수행한다.

이번 절에서는 Jenkins가 특정 프로젝트 코드를 내려받고 Checkout하는 스크립트를 작성한다.

![image](https://user-images.githubusercontent.com/56240505/131523623-94af2c6d-1080-46ef-b387-dd72e7640faa.png)

배포할 프로젝트가 민감한 정보를 서브 모듈로 관리하고 있는 경우 추가적인 설정이 필요하다.

* 서브 모듈에 접근할 수 있는 GitHub 계정의 Personal Access Token이 필요하다.
  * Jenkins가 빌드할 프로젝트 코드를 내려받을 때, 해당 Access Token을 통해 서브 모듈 또한 함께 내려받는다.
  * [GitHub Developer Setting](https://github.com/settings/tokens)에서 발급받는다.
* Jenkins 설정에서 Credentials에 ``Username with password`` 형태로 Access Token을 저장한다.
  * Username에는 GitHub 계정명을, Password에 Access Token을 기입한다.
  * ID는 Jenkins 내부에서 식별할 수 있게 작성한다.

<img width="528" alt="스크린샷 2021-08-31 오전 12 04 42" src="https://user-images.githubusercontent.com/56240505/131523974-016f9a59-4b90-4b88-af3e-b718e75d326b.png">

* 하단의 ``Pipeline Syntax``를 클릭한다.
  * 자주 사용하는 스크립트를 일일이 외울 필요 없이 설정에 따라 자동으로 생성해주는 기능이다.

<img width="636" alt="스크린샷 2021-08-31 오전 12 08 39" src="https://user-images.githubusercontent.com/56240505/131525044-9d7f1f22-8781-43b4-b585-b3a751b243be.png">

* 위 사진과 같이 Sample Step을 ``checkout``으로 선택한다.
* Repository URL은 배포할 프로젝트 URL을 기입하며, 빌드할 브랜치는 목표에 맞게 작성해준다.
* Credentials는 방금 등록한 Access Token을 선택한다.

<img width="474" alt="스크린샷 2021-08-31 오전 12 09 13" src="https://user-images.githubusercontent.com/56240505/131527521-39e65450-2083-4329-8462-bc42e916f2f4.png">

* ``Additional Behaviours`` 탭에서 ``Advanced sub-modules behaviours``를 선택해, Jenkins가 프로젝트 코드를 내려받을 때 서브 모듈 또한 함께 내려받도록 설정한다.
  * 서브 모듈을 사용하지 않는다면 해당 작업을 스킵해도 좋다.

![image](https://user-images.githubusercontent.com/56240505/131528153-c95abb2c-2104-45b3-8cc9-5c6eb1a36886.png)

* 위 사진처럼 체크해주고, 프로젝트 최상위 Root 위치에서 서브 모듈로의 경로를 명시해준다.
* 이후 하단의 ``Generate Pipeline Script``를 클릭하면 스크립트가 생성되는데, 이를 복사해둔다.

<img width="791" alt="스크린샷 2021-08-31 오전 12 13 22" src="https://user-images.githubusercontent.com/56240505/131528522-65021635-514d-4fdd-af2a-f0d4a5daae96.png">

* 스크립트를 작성한다.

### 5.3. Build Script

<img width="737" alt="스크린샷 2021-08-31 오전 12 22 13" src="https://user-images.githubusercontent.com/56240505/131528662-b646a477-21bb-4f86-be3b-3ba336efb8b5.png">

* Checkout 스크립트를 성공적으로 작성했으면, 그 다음 stage를 작성해준다.
* ``gradlew`` 파일이 있는 위치로 이동한 다음 빌드 스크립트를 실행한다.
  * 프로젝트 최상단에 위치하는 경우 ``dir()`` 명령어가 필요없다.

![스크린샷 2021-08-31 오전 12 27 57](https://user-images.githubusercontent.com/56240505/131529152-4b647985-1cce-4acd-bdc6-62bdc9eefe34.png)

* PR을 생성해 Pipeline 스크립트를 시켜보면, 빌드 및 테스트가 잘 수행됨을 확인할 수 있다.

<br>

## 6. Deploy Script 작성

Jenkins 서버는 현재 다음과 같은 작업을 수행 중이다.

* GitHub 측에서 보내는 Webhook Event를 감지하고 파싱한다.
* 적합한 Event가 발생하면 개발자가 지정한 특정 프로젝트의 코드를 내려받고, 특정 브랜치로 Checkout한다.
* 개발자가 작성한 빌드 스크립트를 실행한다.

이제 다음과 같은 작업이 필요하다.

* 빌드한 JAR 결과물을 Jenkins 서버에서 배포 운영 서버로 전송한다.
* 배포 운영 서버에서 자동으로 해당 JAR 파일을 실행시켜, 클라이언트에게 서비스를 배포한다.

<img width="398" alt="스크린샷 2021-09-01 오전 12 17 22" src="https://user-images.githubusercontent.com/56240505/131529576-b92c83a4-4e4d-478f-b33d-58999262f085.png">

먼저 Publish over SSH 플러그인을 설치한 다음, Jenkins 환경 설정 탭에서 ``Publish over SSH`` 부분을 찾아 위 사진처럼 설정해준다.

* Key : JAR 파일을 전달할 배포 운영 서버의 비밀키.
* Name : Jenkins 서버에서 식별할 배포 운영 서버 이름.
* Hostname : 배포 운영 서버의 주소.
* Username : 배포 운영 서버 내 사용 중인 OS
* Remote Directory : 전송된 파일이 위치할 기본 위치.

<img width="691" alt="스크린샷 2021-08-31 오전 12 59 39" src="https://user-images.githubusercontent.com/56240505/131530444-bd568941-d337-4f43-88cb-e134b3517fa6.png">

Checkout과 마찬가지로 ``Pipeline Syntax``를 통해 배포 스크립트를 작성한다.

* 위 사진과 같이 Sample Step을 ``sshPublisher``로 선택한다.
  * SSH Server 이름은 Jenkins 환경 설정 탭에서 입력한 배포 운영 서버의 Name을 기입한다.
  * Source Files : 빌드 이후 생성된 결과물들.
    * 프로젝트 최상위 Root 위치를 기준으로 경로를 작성한다.
  * Remove prefix : 파일 전송 전에 Source Files에 붙은 경로를 제거할 수 있다.
* ``Generate Pipeline Script``를 통해 생성한 스크립트를 복사해둔다.

<img width="773" alt="스크린샷 2021-08-31 오전 1 20 02" src="https://user-images.githubusercontent.com/56240505/131531327-1aeec69d-3808-4e66-9a41-17ff124e8943.png">

Deploy stage에 스크립트를 붙여넣는다.

* 복붙한 스크립트는 배포 이후 수행할 작업에 대한 명령어가 ``execCommand: ''`` 로 공란인 상태다.
  * 파일 전송 이후 JAR 파일 실행 등 수행되어야 할 명령어를 해당 부분에 기입한다.

> Shell

```groovy
execCommand: '''
               PROC=`ps aux | grep blog`
               if [[ $PROC == *"blog"* ]]; then
                 echo "Process is running."
                 kill -9 `ps -ef | grep blog | grep -v grep | awk \'{print $2}\'`
               else
                 echo "Process is not running."
               fi

               FILE_NAME=`ls *SNAPSHOT.jar`
               nohup java -Dspring.profiles.active=prod -jar $FILE_NAME > blog.out 2> blog.err < /dev/null &
            '''
```

* 현재 배포 운영 서버에 동일한 이름의 프로젝트가 실행 중인지 프로세스를 확인한다.
  * 실행 중이라면 해당 프로세스를 제거한다.
  * 리눅스 명령어 및 쉘 스크립트 작성은 별도로 학습한다.
* 이후 Jenkins 서버로부터 전달받은 JAR 파일을 ``nohup``으로 실행한다.

![image](https://user-images.githubusercontent.com/56240505/131532786-2982b1a6-35a5-433a-a9f1-7fb4de23acca.png)

* Jenkins Pipeline을 실행한 뒤, 배포 운영 서버 EC2로 접속해본다.
* Jenkins 서버가 전송한 JAR 파일이 존재하며, 웹 어플리케이션이 실행 중임을 확인할 수 있다.

<br>

## 7. 기타

> Pipeline Script

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: true, recursiveSubmodules: true, reference: 'backend/submodule', trackingSubmodules: true]], userRemoteConfigs: [[credentialsId: '토큰id', url: '프로젝트url']]])
            }
        }

        stage('Build & Test') {
            steps {
                dir('./backend') {
                    sh '''
                      ./gradlew clean bootJar build
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'xlffm3-prod', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
               PROC=`ps aux | grep blog`
               if [[ $PROC == *"blog"* ]]; then
                 echo "Process is running."
                 kill -9 `ps -ef | grep blog | grep -v grep | awk \'{print $2}\'`
               else
                 echo "Process is not running."
               fi

               FILE_NAME=`ls *SNAPSHOT.jar`
               nohup java -Dspring.profiles.active=prod -jar $FILE_NAME > blog.out 2> blog.err < /dev/null &''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: 'backend/build/libs/', sourceFiles: 'backend/build/libs/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
```

* 최종 스크립트는 위와 같다.

> Shell

```bash
sudo apt update && sudo apt install default-jdk

find ./* -name "*jar"

nohup java -jar [jar파일명] 1> [로그파일명] 2>&1  &
```

* 물론 배포 운영 서버에 미리 JDK를 설치해두어야 한다.

<br>

---

## References

* Special Thanks to [손너잘](https://github.com/bperhaps)
* [[CI/CD] Jenkins vs GitLabCI vs Travis](https://owin2828.github.io/devlog/2020/01/09/cicd-1.html)
* [How to set up github webhook trigger on pushing in certain branch](https://stackoverflow.com/questions/58100199/how-to-set-up-github-webhook-trigger-on-pushing-in-certain-branch)
* [Generic Webhook Trigger](https://plugins.jenkins.io/generic-webhook-trigger/)
* [Java Jayway JsonPath 사용법](https://advenoh.tistory.com/28)
* [Scripted Pipeline syntax - Jenkins](https://www.jenkins.io/doc/book/pipeline/)
* [Jenkins pipeline git command submodule update](https://stackoverflow.com/questions/42290133/jenkins-pipeline-git-command-submodule-update)
