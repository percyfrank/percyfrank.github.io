---
title:  "[Spring Boot 개념과 활용] 19장 : MySQL"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-12T08:10:00-05:00
---

## 1. DBCP

* Database Connection Pool.
  * HikariCP (기본).
  * Tomcat CP.
  * Commons DBCP2.
* 커넥션 객체의 개수 및 유지 시간 등에 관련된 설정이다.
* 어플리케이션 성능에 중요하다.

<br>

## 2. DBCP 설정

* 세부 설정은 레퍼런스를 참조하자.
  * ``spring.datasource.hikari.maximum-pool-size=4``
* spring.datasource.hikari.*
* spring.datasource.tomcat.*
* spring.datasource.dbcp2.*

<br>

## 3. MySQL 설정

> Docker

```sh
docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e
MYSQL_DATABASE=springboot -e MYSQL_USER=jipark -e
MYSQL_PASSWORD=pass -d mysql

docker exec -i -t mysql_boot bash
mysql -u root -p
```

* MySQL 커넥터 의존성 추가.
  * mysql-connector-java.
* Docker로 간단하게 mysql 사용하기.

<br>

## 4. MySQL용 Datasource 설정

> application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=jipark
spring.datasource.password=pass
```

* MySQL 라이센스(GPL)를 주의한다.
  * MySQL 대신 MariaDB 사용을 검토한다.
  * 소스 코드 공개 의무 여부를 확인한다.
* Windows 경우 ToolBox를 사용한다면 localhost를 ip 주소로 변경해야 한다.
* 버전에 따라 ssl 등의 옵션을 추가할 수도 있다.

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
