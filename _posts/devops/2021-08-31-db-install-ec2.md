---
title: "AWS EC2 Docker를 활용한 MariaDB 설정"
excerpt: "WAS 서버와 DB 서버를 분리해보자."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-08-31
last_modified_at: 2021-08-31
---

## 1. DB 서버 분리

간단한 토이 프로젝트의 경우, DB를 WAS 서버가 위치한 인스턴스에 함께 설치하더라도 큰 문제가 없다. 반면 배포하려는 서비스를 장기적인 플랜을 가지고 운영할 계획이라면, 하나의 인스턴스에 모든 인프라 요소들을 배치하는 것은 바람직하지 못하다. 트래픽이 해당 인스턴스에 과도하게 집중되며, 인스턴스에 문제가 발생하면 서비스뿐만 내부에서 함께 동작하는 다양한 인프라 서비스에 접근할 수 없게 된다.

인프라 설계 또한 항상 확장 가능성에 열려 있어야 한다. 최근 진행 중인 팀 프로젝트의 경우 로드 밸런싱 도입을 위해 DB 서버가 서비스 WAS로부터 분리될 필요성을 느꼈다.

<br>

## 2. DB 벤더 선택

1. MySQL : 영리 목적으로 사용하려면 유료 버전을 이용해야한다.
2. MariaDB : 상용 버전이 별도로 존재하지 않는 등 라이센스 측면에서 자유롭고 MySQL와 호환이 잘 된다.
3. PostgreSQL : 기존의 MySQL 문법에 익숙한 팀원들에게 러닝 커브가 존재하며, 레퍼런스가 상대적으로 부실하다.

MariaDB는 MySQL과 사실상 사용 방법이 동일하기에 러닝 커브 측면에서 팀원들에게 가장 유리한 벤더라고 판단했다. PostgreSQL은 병렬 상태에 따른 성능 및 복잡한 쿼리 성능 등을 아직 고려할 필요성을 느끼지 못해 후보에서 제외했다.

<br>

## 3. EC2 Docker 설치 및 MariaDB 컨테이너 실행

> Shell

```bash
$ sudo apt update && sudo apt install -y docker.io && sudo apt install -y docker-compose
$ sudo usermod -aG docker ubuntu
```

* ``docker-compose.yml`` 파일을 작성한다.
* 개별적인 명령어를 통해 이미지를 받고 실행할 수 있으나, 파일로 관리하는 것이 편하다.

> docker-compose.yml

```bash
version: '3'
services:
   mariadb:
      image: mariadb:latest
      container_name: mariadb-conatiner
      ports:
         - 3306:3306
      environment:
         MYSQL_DATABASE: testdb
         MYSQL_ROOT_PASSWORD: root
         TZ: Asia/Seoul
      volumes:
         - /home/mariadb:/var/lib/mysql
```

* 설정은 취향에 따라 커스터마이징하면 되며, ``docker-compose.yml`` 작성과 관련된 설정은 하단 레퍼런스를 참조하길 바란다.

> Shell

```bash
$ sudo chown 1000 /home/mariadb
$ docker-compose up -d
```

* 볼륨 마운팅할 디렉토리를 ``/home/mariadb``로 두었는데, 해당 디렉토리에 적절한 권한을 부여해야 컨테이너가 정상 실행된다.
* 권한 부여 후 컨테이너를 실행한다.

> Shell

```bash
$ docker exec -it mariadb-conatiner /bin/bash
$ mysql -uroot -p
```

* ``exec`` 명령어를 통해 컨테이너 내부에 접속하고 root 계정으로 로그인한다.

> Shell

```sql
mysql> create user 'username'@'%' identified by 'password';
mysql> grant all privileges on *.* to 'username'@'%';
mysql> flush privileges;
mysql> CREATE DATABASE db_name DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

* 외부 WAS에서 사용할 사용자 계정을 생성하고 권한을 부여한다.
* 이후 사용할 DB를 생성해준다.

> application-prod.yml

```yml
spring:
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://{db-ec2-private-ip}:{port}/{db-name}?serverTimezone=UTC&characterEncoding=UTF-8
    username: {username}
    password: {password}
```

* 웹 어플리케이션의 DB 설정 관련 정보를 적합하게 수정한다.
  * ``db-name``은 root 계정 로그인 뒤 생성한 DB 이름을 의미한다.
  * 뒤의 Suffix는 DB 설정에 알맞게 가져간다.
* 웹 어플리케이션을 실행해 DB와 잘 연결되는지 확인한다.

> Shell

```sql
mysql> SELECT Host,User,plugin,authentication_string FROM mysql.user;
```

* 만약 DB 연결에 실패한다면 위 명령어를 통해 해당 사용자 계정에 대해 적절한 권한이 부여되었는지 확인한다.
* 사용자 계정의 Host가 localhost라면 자기 자신의 machine에만 한정된 범위의 접근 권한이다.

> Shell

```sql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'username'@'%' IDENTIFIED BY 'password';
```

* 위 옵션을 준다면 Host가 ``%``로 나오며, 이는 모든 IP에 대한 접근을 가능하게 한다.
* 변경 사항을 flush하고 다시금 WAS를 실행해보면 DB가 잘 연결됨을 확인할 수 있다.

<br>

---

## References

* [docker-compose란?](https://conanglog.tistory.com/72)
* [mariaDB(mysql) 원격 접속 허용하기 (How to allow 'Remote access' for mariaDB(mysql)) & docker로 mariadb 실행하기위한 docker-compose.yml & custom.cnf](https://4urdev.tistory.com/82)
