---
title: "Docker Container 이해"
excerpt: "우아한테크코스 배포 인프라 미션 학습 내용입니다."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-06-21
last_modified_at: 2021-06-21
---

## 1. Snowflake Server

애플리케이션이 배포되는 환경을 서버에 직접 설치하여 구성할 경우 Snowflake Server 이슈를 직면하기 쉽다. 직접 서버를 설치하고 OS를 인스톨한 뒤, 필요한 소프트웨어와 애플리케이션을 설치하여 운영한다. 문제가 생기면 패치 등을 통해 서버에 지속적으로 적용하고, 어플리케이션은 CI/CD 툴로 배포한다.

* 한 번 설치한 서버에 계속해서 설정을 변경하고 패치를 적용하는 등의 업데이트를 지속하면, 해당 서버의 설정을 추후 다시 똑같이 설정하기 어렵다.
* 여러 대의 웹 서버를 운영하는데, 특정 서버를 패치하면 다른 동일한 웹 서버를 모두 패치하지 않으면 구성이 달라진다.
* 기존에 사용하던 서버와 새로 구축한 서버간에 설정의 차이가 발생하면 운영 이슈로 이어진다.

문서화가 아무리 잘 되어있어도, 담당자가 바뀌면 이력이 제대로 유지되기 어렵다. 또한 장비를 업그레이드 하거나 OS를 새로 인스톨해서 같은 환경을 꾸미고자 할때 예전 환경과 동일한 환경을 구성하기가 어렵고, 장애가 나기 쉽다.

이처럼 한번 설정하면 다시 설정이 불가능한, “마치 눈처럼 녹아버리는" 서버의 형태를 스노우 플레이크 서버라고 한다.

### 1.1. 가상화

가상화란 물리적인 컴포넌트(HW 장치)를 논리적인 객체로 추상화 하는 것을 의미한다. 하드웨어에 종속된 컴퓨터 리소스를 추상화하여 서버, 스토리지, 네트워크 등의 소프트웨어 IT 서비스를 생성하는 솔루션을 일컫는다.

![image](https://user-images.githubusercontent.com/56240505/132978241-2a76eb8a-4647-4bb9-8a2f-2a01e860d881.png)

* 용도가 다른 세 대의 물리적인 서버가 존재한다고 가정해보자.
* 각각의 서버는 주어진 책임을 수행하는데 잠재적인 실행 용량에 비해 30%의 용량만 사용하고 있다.

![image](https://user-images.githubusercontent.com/56240505/132978266-4c2e6853-d74b-4187-a7a1-cb64dbc2bdd5.png)

* 가상화를 통해 메일 서버를 2개의 고유한 서버로 분리해, 독리적인 태스크를 처리하고 레거시 어플리케이션을 마이그레이션할 수 있다.
  * 하드웨어를 더 효율적으로 사용할 수 있다.
* 즉, 가상화란 마치 하나의 장치를 여러개처럼 동작하게 하는 것이다.
  * 여러 장치를 묶어 마치 하나의 장치인 것처럼 사용자에게 공유자원으로 제공한다.
  * 하나의 물리적 서버에서 여러 운영체제와 애플리케이션을 실행할 수 있도록 하는 소프트웨어 기술을 의미한다.

![image](https://user-images.githubusercontent.com/56240505/132978347-b8cf4adb-f9b4-4458-8613-0b7fb171c782.png)

* 가상화의 대상이 되는 컴퓨팅 자원은 프로세서(CPU), 메모리(Memory), 스토리지(Storage), 네트워크(Network)를 포함한다.
* 이들을 쪼개거나 합쳐서 자원을 더욱 더 효율적으로 사용하고, 분산처리를 가능하게 한다.

### 1.2. 기존의 OS 가상화 방식

![image](https://user-images.githubusercontent.com/56240505/124621531-51ef0d80-deb5-11eb-81ab-08c19e0e1ff1.png)

기존의 가상화 기술은 하이퍼바이저를 이용해 여러 개의 OS를 하나의 호스트에서 생성해 사용하는 방식이다. 하이퍼바이저는 호스트 컴퓨터에서 다수의 운영체제를 동시에 실행하기 위한 논리적 플랫폼이다.

* Guest OS에서 동작하는 어플리케이션은 일반적으로 Host OS 바로 위에서 동작하는 어플리케이션보다 성능이 떨어진다.
* 각종 시스템 자원을 가상화하고 독립된 공간을 생성하는 등 작업이 하이퍼바이저를 거쳐야하기 때문이다.
* 어플리케이션과 기기까지 도달하는 중간 액세스 포인트가 많아 메모리 접근 및 파일 접근 등의 속도가 느리다.

![image](https://user-images.githubusercontent.com/56240505/132978458-96e42b88-521c-4926-bd3e-48c0e36f5c50.png)

하이퍼바이저는 하이퍼바이저는 가상 머신(Virtual Machine, VM)을 생성하고 구동하는 소프트웨어다. VM은 컴퓨터 시스템을 에뮬레이션함으로써 컴퓨터 안에서 또 다른 컴퓨터를 동작시키는 것이다. 가상 머신은 GuestOS를 사용하기 위한 라이브러리 및 커널 등을 전부 포함하기에 가상 머신을 배포하기 위한 이미지 크기가 너무 크다.

* OS 설치에 필요한 자원의 낭비가 심해진다.
* 가상 머신 이미지를 애플리케이션으로 배포하기 힘들다.

### 1.3. 문제점

새로운 서버에서 어플리케이션을 배포하기 위해 다음과 같은 번거로운 작업이 수반된다.

* H/W 장비 구매.
* OS 설치.
* 부팅.
* 각종 서버 설정.
* 어플리케이션 빌드 및 실행.

OS 가상화 방식을 사용하더라도 부팅과 각종 서버 설정 및 재빌드 등의 작업은 생략할 수 없다. 서버를 Scale-Out하다 보면 전형적인 Snowflake 서버 문제가 발생한다.

<br>

## 2. 컨테이너

OS까지 띄우지 않고 특정 환경에 종속되지 않은 상태로 어플리케이션을 띄우는 방법이 바로 컨테이너다. Docker는 격리된 CPU, 메모리, 디스크, 네트워크를 가진 공간인 컨테이너를 만들고, 해당 공간에서 프로세스를 실행해 유저에게 서비스를 제공한다.

* 컨테이너는 자신의 내부 공간에서 관리하는 무언가의 생성과 운영 및 제거 등 생애주기를 관리한다.
* 이에 필요한 것이 바로 **격리**다.
* 컨테이너는 외부와 격리되어 다른 컨테이너의 내부에 영향을 끼치지 않으니, 사용자는 **컨테이너 단위**로 어플리케이션을 적재 및 사고할 수 있다.

### 2.1. 프로세스

프로세스란 실행 중인 프로그램을 뜻하며, 보다 구체적으로 프로그램의 명령 및 실행시에 필요한 정보 조합의 오브젝트다. 즉, 프로그램이 OS에 실행되는 단위를 뜻한다.

* HDD 등 저장 장치에 있는 프로그램을 실행하면 실행할 때의 명령 및 실행에 필요한 정보 등이 메모리에 적재된다.
* 1개의 CPU는 동시에 1개의 프로세스 작업을 수행할 수 있으며, 프로세스를 종료하면 메모리 등의 자원을 반납한다.

### 2.2. 프로세스 추상화

컨테이너는 프로세스를 관리하며, 이는 즉 **프로세스를 추상화**한다는 말과 동일하다. 프로세스가 생성되고 운영되고 제거되기까지 생애주기를 관리하며, 격리가 필요하다. 컨테이너 안의 프로세스는 제한된 자원 내에서 제한된 사용자만 접근할 수 있다.

* chroot : 특정 자원만 사용하도록 제한한다.
* cgroup : 자원의 사용량을 제한한다.
* namespace로 특정 유저만 자원을 볼 수 있도록 제한한다.

![image](https://user-images.githubusercontent.com/56240505/132979512-4a7a92d2-9057-4c3f-912d-374d3432cd36.png)

앞서 언급했듯 Hypervisor 방식은 Guest OS를 위한 시스템 자원을 가상화하고 독립된 공간을 생성하는 작업 등을 거친다. 별도의 OS 설치에 필요한 자원을 요구하며, 어플리케이션과 기기까지 도달하는데 중간 액세스 포인트들이 많아져 성능이 저하된다.

* Docker는 Host OS 수준에서 가상화를 실시하여 다수의 컨테이너를 OS 커널에서 직접 구동하며, 컨테이너는 Host OS의 시스템 커널 및 자원을 직접 사용한다.
* Hypervisor를 통한 OS 부팅보다 컨테이너 시동이 훨씬 빠르며, 메모리를 덜 차지한다.

### 2.3. 장점

이미지란 서비스 운영에 필요한 서버 프로그램, 소스 코드, 컴파일된 실행 파일 등을 묶은 형태다.

* 프로그램(데몬)을 직접 구동해야 한다.
* Docker는 DockerHub를 통 GitHub처럼 이미지를 공유할 수 있다.

컨테이너는 이미지를 실행한 형태로, 이미지로 여러 개의 컨테이너를 만들 수 있다. Docker는 게스트 OS를 설치하지 않는다. 이미지에 서버 운영을 위한 프로그램과 라이브러리만 격리해서 설치하기 때문에 이미지 용량이 Hypervisor 방식보다 크게 줄어든다.

또한 컨테이너 내용을 수정해도 Host OS에 영향을 끼치지 않는다. 어플리케이션 개발 및 배포가 편해지고, 독립성 및 확장성이 높아진다.

요약하자면 컨테이너는 가상화보다 훨씬 가벼운 가상화 기술이며, 리눅스 내부의 컨테이너 기술을 사용해서 가상 공간을 만든다. 만들어진 공간의 실행 파일은 호스트에서 직접 실행하기 때문에, 가상화보다 격리의 개념이 더 가깝다.

<br>

## 3. 파일 시스템

> Shell

```bash
$ sudo apt-get update && \
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
sudo apt-get update && \
sudo apt-get install -y docker-ce && \
sudo usermod -aG docker ubuntu && \
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
sudo chmod +x /usr/local/bin/docker-compose && \
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

$ sudo lsof -c docker
dockerd     880 root  txt       REG              202,1 102066512      55737 /usr/bin/dockerd
dockerd     880 root    6u     unix 0xffff953085804400       0t0      18324 /var/run/docker.sock type=STREAM
```

사용자가 명령어를 입력하면 docker.sock 을 통해 도커 데몬의 API를 호출한다. 컨테이너는 호스트 시스템의 커널을 사용한다. 컨테이너는 이미지에 따라서 실행되는 환경(파일 시스템)이 달라진다.

* docker stats 및 docker system df 명령어를 통해 도커가 현재 Host의 자원을 얼마나 사용하고 있는지 확인할 수 있다.

> Shell

```bash
$ docker run -it ubuntu:latest bash
root@03d7ffae4c3c:/# cat /etc/*-release
DISTRIB_ID=Ubuntu

$ docker run -it centos:latest bash
[root@6ec49524540c /]# cat /etc/*-release
CentOS Linux release 8.2.2004 (Core)
```

컨테이너가 서로 다른 파일 시스템을 가질 수 있는 이유는 무엇일까? chroot를 활용하여 이미지(파일의 집합)를 루트 파일 시스템으로 강제로 인식시켜 프로세스를 실행하기 때문이다. 컨테이너도 결국 프로세스다.

<br>

## 4. Docker 이미지

> Shell

```bash
$ docker pull ubuntu:latest
```

이미지는 파일들의 집합이고, 컨테이너는 이 파일들의 집합 위에서 실행된 특별한 프로세스다. 프로세스의 데이터가 변경되더라도 원본 프로그램 이미지를 변경할 수 없듯, 컨테이너의 데이터가 변경되더라도 컨테이너 이미지의 내용을 변경할 수 없다.

* 도커의 이미지는 [저장소 이름]/[이미지 이름]:[태그]의 형태로 구성된다.

<br>

## 5. Docker 네트워크

![image](https://user-images.githubusercontent.com/56240505/124627343-6eda0f80-deba-11eb-805f-e3abcc05b300.png)

1. 컨테이너는 생성될 때 namespace로 격리되고, 통신을 위한 네트워크 인터페이스(eth0)를 할당받는다.
  * 컨테이너는 각각 고유한 Mac 주소 및 IP가 할당된다.
2. Host에는 veth 인터페이스가 생성되고 컨테이너 내의 eth0과 연결된다.
3. Host의 veth 인터페이스들은 docker0이라는 다른 인터페이스와 연결된다.
  * docker0은 Docker 실행 시 자동으로 생성되는 가상의 브릿지다.
4. 컨테이너의 내부 라우팅 테이블에 따르면 모르는 대역의 경우, docker0으로 라우팅하도록 되어있다.
  * 컨테이너들의 veth들은 docker0릍 통해 컨테이너들간에 통신이 가능하다.
  * 컨테이너는 gateway인 docker0를 거쳐 외부와 통신한다.

### 5.1. 포트 포워딩

외부 사용자는 컨테이너의 IP를 알기 어렵다. 또한 외부 요청은 컨테이너를 향해 직접 오는 것이 아니라, Host의 물리적인 네트워크 인터페이스로 온다. 따라서 포트 포워딩 옵션을 이용한다.

* ``docker run -p 80:80 -p 443:443``

Host OS의 n번 포트 번호를 사용하는 소켓 파일과 컨테이너 내의 n번 포트 번호를 사용하는 소켓 파일을 연결해준다. Host OS가 직접 요청을 처리할 수 있게 된다.

<br>

## 6. Docker 볼륨

이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며, 컨테이너의 변경 사항만 별도로 저장해서 각 컨테이너의 정보를 보존한다. 그러나 mysql과 같이 컨테이너 계층에 저장돼 있던 데이터베이스의 정보를 삭제해서는 안되는 경우가 존재한다. 컨테이너 데이터를 영속적인 데이터로 활용하기 위한 방법 중 ``-v`` 볼륨 옵션을 활용할 수 있다.

* 컨테이너가 아닌 외부에 데이터를 저장하고 컨테이너는 그 데이터로 동작하도록 설계(stateless)해야 한다.
* 컨테이너 자체는 상태가 없고 상태를 결정하는 데이터는 외부로부터 제공받도록 구성하도록 한다.

<br>

---

## References

* 우아한테크코스
* [가상화(Virtualization)란 무엇일까요?](https://www.redhat.com/ko/topics/virtualization/what-is-virtualization)
* [[스터디 정리] 하이퍼바이저의 종류](https://lovejaco.github.io/posts/two-types-of-hypervisors/)
