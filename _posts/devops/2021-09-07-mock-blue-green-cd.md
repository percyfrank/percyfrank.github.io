---
title: "Jenkins & Nginx를 활용한 Blue/Green 무중단 배포 구현"
excerpt: "AWS Beanstalk 및 Auto Scaling 없이 무중단 배포를 흉내내보자."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-09-07
last_modified_at: 2021-09-07
---

## 1. 계기

팀 프로젝트에서 **무중단 배포를 구현했으면 좋겠다**는 의견이 나왔다. Blue/Green 무중단 배포를 위한 AWS 기능들을 살펴보았다.

* AWS Beanstalk, AWS Auto Scaling, AWS Code Deploy 등.

문제는 우아한테크코스에서 제공하는 AWS 계정은 해당 기능들의 사용 권한이 없었다. 따라서 직접 Blue/Green 무중단 배포를 구현하기로 결정했다.

<br>

## 2. 참고 서적

이동욱님의 [스프링 부트와 AWS로 혼자 구현하는 웹 서비스 (A.K.A 보라돌이)](http://www.yes24.com/Product/Goods/83849117) 책에서는 무중단 배포를 단 1개의 EC2 인스턴스로 간단하게 구현한다.

1. EC2 1대에 Nginx 및 JDK를 설치해둔다.
2. CI 툴을 통해 JAR 파일이 EC2로 전송되면, 미리 준비된 EC2 내부의 쉘 스크립트를 실행한다.
3. 해당 쉘 스크립트를 통해 현재 81 포트에 실행 중인 JAR가 있는지 확인한다.
  * 없다면 JAR를 81포트로 실행한다.
    * EC2 80 포트로 들어온 요청을 81 포트로 전달하도록 Nginx 설정 파일을 수정하고 Reload한다.
    * 기존 82 포트에서 실행 중이던 JAR 프로세스를 죽인다.
  * 있다면 JAR를 82 포트로 실행한다.
    * EC2 80 포트로 들어온 요청을 82 포트로 전달하도록 Nginx 설정 파일을 수정하고 Reload한다.
    * 기존 81 포트에서 실행 중이던 JAR 프로세스를 종료한다.
4. 신규 코드 배포가 발생할 때마다 JAR 실행 포트가 ``81 -> 82 -> 81 ...`` 식으로 스위칭된다.

해당 서적의 원리를 차용해서 Blue/Green 배포 환경을 조성하려고 한다.

### 2.1. 회의 결과

시간이 넉넉하다면 쿠버네티스 등을 학습해서 적용했겠지만, 현실적인 이유로 반려했다. 최종적으로 참고 서적과 유사하게 EC2 인스턴스 IP 포워딩 방식(?)으로 무중단 배포를 구현하기로 결정했다. 우아한테크코스 AWS 계정의 경우, EC2 생성 개수 제한이 없는 등 비용적인 문제에서 자유로웠기 때문이다.

현재 운영에 사용 중인 WAS EC2 개수만큼 추가적으로 WAS EC2를 생성하고 이들을 2개 그룹(Blue/Green)으로 나눈 뒤, 배포 때마다 Nginx 리버스 프록시가 참조하는 운영 EC2 인스턴스의 IP를 스위칭한다.

<br>

## 3. 원리

1. Blue/Green 모방을 위해 운영 WAS EC2를 2대 더 생성해 총 4대를 만들고, 2대씩 Group A와 Group B로 나눈다.
2. Jenkins 파이프라인은 빌드 및 테스트 이후, Group A에 속한 WAS들의 80포트가 살아있는지 확인한다.
  * 살아있다면 Group B가 신규 배포 대상 그룹(Green)이 되며, Group A는 배포 완료 후 80 포트 JAR 프로세스를 종료할 대상 그룹(Blue)이 된다.
  * 살아있지 않다면 Group A가 신규 배포 대상 그룹(Green)이 되며, Group B는 배포 완료 후 80 포트 JAR 프로세스를 종료할 대상 그룹(Blue)이 된다.
3. Green에 속한 WAS 서버들에게 JAR 파일을 전송하고 80포트로 실행하도록 한다.
4. Nginx EC2 80 포트로 들어온 요청을 Green 그룹 WAS들로 전달하도록 Nginx 설정 파일을 수정 및 Reload한다.
5. Blue에 속한 WAS 서버들의 80 포트 JAR 프로세스를 종료한다.
6. 신규 코드 배포가 발생할 때마다 실제 서비스를 배포하는 그룹이 ``Group A -> Group B -> Group A ...`` 식으로 스위칭된다.

<br>

## 4. 예제

> [Jenkins 서버 및 Publish Over SSH 플러그인](https://xlffm3.github.io/devops/jenkins-pipeline/)에 대한 사용법 숙지가 선행되어야 한다.

<img width="393" alt="스크린샷 2021-09-08 오전 12 00 46" src="https://user-images.githubusercontent.com/56240505/132367186-f2091cae-b737-42fd-b027-952561d855fe.png">

* Blue/Green 배포 환경에 해당하는 Group A 및 Group B WAS 서버 주소들을 Jenkins 환경 변수 설정에 추가한다.
* **Group A 및 Group B에 속한 모든 WAS 서버 주소들은 [Publish Over SSH](https://xlffm3.github.io/devops/jenkins-pipeline/#6-deploy-script-%EC%9E%91%EC%84%B1) 설정에도 모두 등록되어야 한다.**

### 4.1. 전체 파이프라인 코드

> Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                //중략
            }
        }

        stage('Build & Test') {
            steps {
                //중략
            }
        }

        stage('Deploy') {
            environment {
                DEPLOY_GROUP= sh (returnStdout: true, script: '''#!/bin/bash
                    group_a_array=($group_a)
                    target_group=''
                    alive_was_count=0
                    for i in "${group_a_array[@]}"
                      do
                        if curl -I -X OPTIONS "http://${i}:8080"; then
                          alive_was_count=$((alive_was_count+1))
                        fi
                      done
                    if [ ${alive_was_count} -eq 0 ];then
                        echo 'group_a'
                    elif [ ${alive_was_count} -eq ${#group_a_array[@]} ];then
                        echo 'group_b'
                    else
                        echo 'currently runnig was counts are not correct'
                        exit 100
                    fi''').trim()
            }

            steps {
                script {
                    def deploy_server_ips;
                    def close_server_ips;
                    def envVar = env.getEnvironment();
                    if (DEPLOY_GROUP == "group_a") {
                        deploy_server_ips = envVar.group_a.split(' ');
                        close_server_ips = envVar.group_b.split(' ');
                    } else {
                        deploy_server_ips = envVar.group_b.split(' ');
                        close_server_ips = envVar.group_a.split(' ');
                    }
                    def deploy_server_ips_inline = deploy_server_ips.join(' ');
                    def map = parseSSHServerConfiguration();

                    for (item in deploy_server_ips) {
                        sshPublisher(publishers: [sshPublisherDesc(configName: map.get(item), transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
                           FILE_NAME=`ls *SNAPSHOT.jar`
                           nohup java -Dspring.profiles.active=prod -jar $FILE_NAME > blog.out 2> blog.err < /dev/null &''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: 'backend/build/libs/', sourceFiles: 'backend/build/libs/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                    }

                    sshPublisher(publishers: [sshPublisherDesc(configName: 'xlffm3-reverse-proxy', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''#!/bin/bash
                        function parse_was_address() {
                        	array=(''' +deploy_server_ips_inline+ ''')
                          str=""
                        	for i in "${array[@]}"
                                do
                        	        str+="		server ${i}:8080;\n"
                                done
                        	echo -e "$str"
                        }
                        rm -rf nginx.conf
                        cat template_header >> nginx.conf && echo -e "$(parse_was_address)" >> nginx.conf && cat template_footer >> nginx.conf
                        sudo \\cp -f nginx.conf /etc/nginx/
                        sudo service nginx reload
                        ''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])

                    for (item in close_server_ips) {
                        sshPublisher(publishers: [sshPublisherDesc(configName: map.get(item), transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
                            PROC=`ps aux | grep blog`
                            if [[ $PROC == *"blog"* ]]; then
                                echo "Process is running."
                                kill -9 `ps -ef | grep blog | grep -v grep | awk \'{print $2}\'`
                            else
                                echo "Process is not running."
                            fi''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                    }
                }
            }
        }
    }
}

@NonCPS
def parseSSHServerConfiguration() {
    def xml = new XmlSlurper().parse("${JENKINS_HOME}/jenkins.plugins.publish_over_ssh.BapSshPublisherPlugin.xml");
    def server_nick_names = xml.'**'.findAll{it.name() == 'name'};
    def server_ip_address = xml.'**'.findAll{it.name() == 'hostname'};
    def map = new HashMap();
    for (i = 0; i < server_nick_names.size(); i++) {
        def server_ip_str = (String) server_ip_address.get(i);
        def server_nick_name_str = (String) server_nick_names.get(i);
        map.put(server_ip_str, server_nick_name_str);
    }
    return map;
}
```

* 세세한 쉘 스크립트 코드 및 Groovy 문법은 개별 학습하길 바란다.

### 4.2. 배포 대상 그룹 탐색

```groovy
environment {
    DEPLOY_GROUP= sh (returnStdout: true, script: '''#!/bin/bash
        group_a_array=($group_a)
        target_group=''
        alive_was_count=0
        for i in "${group_a_array[@]}"
          do
            if curl -I -X OPTIONS "http://${i}:8080"; then
              alive_was_count=$((alive_was_count+1))
            fi
          done
        if [ ${alive_was_count} -eq 0 ];then
            echo 'group_a'
        elif [ ${alive_was_count} -eq ${#group_a_array[@]} ];then
            echo 'group_b'
        else
            echo 'currently runnig was counts are not correct'
            exit 100
        fi''').trim()
}
```

* 먼저 Group A에 속한 WAS 주소(Jenkins 환경 변수)를 배열에 담는다.
* 배열을 순회하면서 8080포트로 OPTIONS 요청을 보냄으로써, Group A에 속한 WAS들이 80포트로 동작 중인지 확인한다.
* 결과를 종합해 배포 대상 그룹을 선정한다.
* ``echo`` 명령어로 배포 대상 그룹을 반환해 DEPLOY_GROUP이라는 환경 변수에 담는다.
  * 쉘 스크립트 함수가 반환하는 값은 외부에서는 접근이 불가능하지만, ``environment`` 블럭 내부의 환경 변수에 정의해두면 외부의 Groovy 스크립트에서도 값을 이용할 수 있다.

### 4.3. 그룹별 IP 배열 생성

```groovy
def deploy_server_ips;
def close_server_ips;
def envVar = env.getEnvironment();
if (DEPLOY_GROUP == "group_a") {
    deploy_server_ips = envVar.group_a.split(' ');
    close_server_ips = envVar.group_b.split(' ');
} else {
    deploy_server_ips = envVar.group_b.split(' ');
    close_server_ips = envVar.group_a.split(' ');
}
```

* Group A 및 Group B 환경 변수는 공백으로 구분된 여러 WAS IP 문자열이다.
  * Split을 통해 생성한 배열을 조건에 따라 적합한 변수에 담아둔다.

### 4.4. IP에 해당하는 SSH Server Name 탐색

![image](https://user-images.githubusercontent.com/56240505/132369785-e302c9ff-bc56-47bf-a90c-0d7c35b9fa21.png)

Publish Over SSH로 JAR 파일을 전송하거나 특정 명령어를 실행하는데, 이 때 IP(Hostname)이 아닌 별칭(Name)이 필요하다. 현재 우리는 Group A 및 Group B에 속한 WAS의 Hostname은 알아도 Name은 모른다.

* Jenkins 환경 설정에서 Publish Over SSH에 배포할 서버 정보(Hostname, Name 등)을 설정해두었다면, 해당 정보는 ``${JENKINS_HOME}/jenkins.plugins.publish_over_ssh.BapSshPublisherPlugin.xml``에 저장된다.
* 이를 파싱하여 IP(Hostname)를 Key로, 별칭(Name)을 Value로 하는 Map을 생성해 무중단 배포 스크립트에서 활용한다.

```groovy
def map = parseSSHServerConfiguration();

//... 중략

@NonCPS
def parseSSHServerConfiguration() {
    def xml = new XmlSlurper().parse("${JENKINS_HOME}/jenkins.plugins.publish_over_ssh.BapSshPublisherPlugin.xml");
    def server_nick_names = xml.'**'.findAll{it.name() == 'name'};
    def server_ip_address = xml.'**'.findAll{it.name() == 'hostname'};
    def map = new HashMap();
    for (i = 0; i < server_nick_names.size(); i++) {
        def server_ip_str = (String) server_ip_address.get(i);
        def server_nick_name_str = (String) server_nick_names.get(i);
        map.put(server_ip_str, server_nick_name_str);
    }
    return map;
}
```

XmlSlurper로 파싱해서 얻은 데이터의 자료형은 ``List<String>``이 아니라 ``List<ChildNode>``이기 때문에, 적절히 타입 캐스팅한다. @NonCPS를 부착한 별도의 메서드로 로직을 분리한 이유는 다음과 같다.

* 기본적으로 파이프라인 스크립트 내부에서 생성 및 사용하는 오브젝트들은 직렬화가 가능해야한다.
  * Jenkins는 작업 중지 및 재개 등을 위해 스크립트의 스텝 별 상태를 직렬화해 저장한다.
* XML 파싱 관련 객체들은 직렬화가 불가능해서 파이프라인을 실행하면 에러가 발생한다.
* 해당 애너테이션이 부착된 함수는 직렬화 및 저장 대상에서 벗어난다.

### 4.5. 신규 배포 대상 그룹(Green) : JAR 배포

```groovy
for (item in deploy_server_ips) {
    sshPublisher(publishers: [sshPublisherDesc(configName: map.get(item), transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
       FILE_NAME=`ls *SNAPSHOT.jar`
       nohup java -Dspring.profiles.active=prod -jar $FILE_NAME > blog.out 2> blog.err < /dev/null &''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: 'backend/build/libs/', sourceFiles: 'backend/build/libs/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
}
```

반복문을 순회하면서 신규 배포 대상 그룹에 속한 WAS 서버로 JAR 파일을 전송하고 실행한다.

* sshPublisher를 사용할 때 ``configName``에는 배포 대상 서버의 IP(Hostname)이 아닌 별칭(Name)을 기입해야 한다.
* 이 때, 미리 만들어둔 Map에서 배포 대상 WAS IP에 해당하는 별칭을 조회한다.

### 4.6. Nginx 설정 파일 수정 및 Reload

> nginx.conf

```
events {}

http {       
  upstream app {
    server {첫 번째 운영 서버 EC2 private IP}:8080;
    server {두 번째 운영 서버 EC2 private IP}:8080;
    # 추후 증설 가능
  }

  # Redirect all traffic to HTTPS
  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl;
    # 기타 설정 중략

    location / {
      proxy_pass http://app;    
    }
  }
}
```

* 리버스 프록시 서버에서 사용하는 Nginx 설정 파일 구조는 위와 같다.
* 로드 밸런싱을 사용한다면, Nginx 설정 파일에도 사용 중인 배포 WAS 서버 주소들을 ``upstream`` 블락에 전부 기입해야 한다.

> template_header

```
events {}

http {       
    upstream app {
```

> template_footer

```
    # 추후 증설 가능
    }

    # Redirect all traffic to HTTPS
    server {
    listen 80;
    return 301 https://$host$request_uri;
    }

    server {
    listen 443 ssl;
    # 기타 설정 중략

    location / {
      proxy_pass http://app;    
    }
  }
}
```

* 기입해야 할 배포 WAS 주소의 개수는 로드 밸런싱 상태에 따라 동적으로 변할 수 있기 때문에, 최대한 자동화를 해보자.
* 먼저 upstream 블락을 기준으로 위 및 아래 내용을 분리하고, 리버스 프록시 EC2에 별도의 파일로 저장해둔다.

```groovy
def deploy_server_ips_inline = deploy_server_ips.join(' ');

sshPublisher(publishers: [sshPublisherDesc(configName: 'xlffm3-reverse-proxy', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''#!/bin/bash
    function parse_was_address() {
      array=(''' +deploy_server_ips_inline+ ''')
      str=""
      for i in "${array[@]}"
            do
              str+="		server ${i}:8080;\n"
            done
      echo -e "$str"
    }
    rm -rf nginx.conf
    cat template_header >> nginx.conf && echo -e "$(parse_was_address)" >> nginx.conf && cat template_footer >> nginx.conf
    sudo \\cp -f nginx.conf /etc/nginx/
    sudo service nginx reload
    ''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
```

> **주의) Nginx Reload 전에 Green 그룹의 WAS들이 완벽하게 배포될 때 까지 반복문을 돌며 Health Check하는 코드가 누락되었다.**

* ``deploy_server_ips_inline``은 공백으로 구분된 Green 그룹에 속한 WAS IP 문자열이다.
* ``array=(''' +deploy_server_ips_inline+ ''')``는 Green 그룹에 속한 WAS IP들의 배열이다.
* ``parse_was_address()`` 함수는 리버스 프록시 서버 내부에서 주어진 IP 배열을 순회하면서, ``server {was ip}:8080\n`` 형태의 문자열을 생성하고 계속 append한다.
* 리눅스의 리다이렉션을 통해, template_header 내용과 ``parse_was_address()`` 리턴 결과 및 template_footer 내용을 순차적으로 ``nginx.conf`` 파일에 작성함으로써 새로운 설정 파일을 생성한다.
* Nginx 설정 파일을 적합한 위치로 이동시키고, Nginx를 Reload한다.

### 4.7. 제거 대상 그룹(Blue) : 실행 중인 프로세스 제거

```groovy
for (item in close_server_ips) {
    sshPublisher(publishers: [sshPublisherDesc(configName: map.get(item), transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''
        PROC=`ps aux | grep blog`
        if [[ $PROC == *"blog"* ]]; then
            echo "Process is running."
            kill -9 `ps -ef | grep blog | grep -v grep | awk \'{print $2}\'`
        else
            echo "Process is not running."
        fi''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
}
```

* 마찬가지로 반복문을 순회하면서 제거 대상 그룹에 속한 WAS 서버로 접속한다.
* 실행 중인 80 포트의 JAR 프로세스를 제거한다.

### 4.8. 추후 WAS를 증설하는 경우

<img width="393" alt="스크린샷 2021-09-08 오전 12 00 46" src="https://user-images.githubusercontent.com/56240505/132367186-f2091cae-b737-42fd-b027-952561d855fe.png">

추후 WAS를 증설하는 경우, Jenkins 환경 변수 설정에 공백을 구분자로 WAS 주소를 추가해준다.

* Group A 및 Group B (Blue/Green)의 IP 개수는 항상 대칭을 이루어야 한다.
* 추가된 WAS 서버 주소들을 Publish Over SSH 설정에도 모두 등록해준다.

<br>

## 5. 한계

연습 차원에서 흉내를 낸 것일 뿐, 이상적인 Blue/Green 무중단 배포는 아니다. AWS Beanstalk 및 AWS Auto Scaling AWS 등을 활용하는 방식과 다르게, 항상 Green 환경의 EC2 인스턴스가 떠있어야하기 때문에 비용이 두 배로 든다.

<br>

---

## References

* [쉘 스크립트 사용법(배열을 사용하는 방법)](https://shlee1990.tistory.com/918)
* [Run bash command on jenkins pipeline](https://stackoverflow.com/questions/44330148/run-bash-command-on-jenkins-pipeline)
* [How to set environment variables in Jenkins?](https://stackoverflow.com/questions/10625259/how-to-set-environment-variables-in-jenkins/53430757#53430757)
* [Accessing Variables Specified for Jenkins Groovy Plugin Script](https://stackoverflow.com/questions/10215959/accessing-variables-specified-for-jenkins-groovy-plugin-script)
* [[Shell Script] 쉘 스크립트 함수나 실행에서 반환값(Return Value) 얻기](https://twpower.github.io/134-how-to-return-shell-scipt-value)
* [What is the effect of @NonCPS in a Jenkins pipeline script](https://stackoverflow.com/questions/42295921/what-is-the-effect-of-noncps-in-a-jenkins-pipeline-script)
* [Jenkins…Access XML Tag value in xml file using Groovy in Jenkins](https://stackoverflow.com/questions/45727256/jenkins-access-xml-tag-value-in-xml-file-using-groovy-in-jenkins)
