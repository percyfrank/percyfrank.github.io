---
title: "Nginx Reverse Proxy TLS 및 로드 밸런싱 적용"
excerpt: "TLS 및 로드 밸런싱을 적용해보자."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-09-01
last_modified_at: 2021-09-01
---

## 1. Reverse Proxy

Reverse Proxy는 클라이언트 요청을 받아서 적절한 웹 서버로 요청을 전송한다. 통상의 Proxy Server는 LAN -> WAN의 요청을 대리로 수행하지만, Reverse Proxy는 WAN -> LAN의 요청을 대리한다.

* 웹 서버는 요청을 처리하고 응답을 Reverse Proxy로 반환한다.
* 요청을 받은 Reverse Proxy는 그 응답을 클라이언트로 반환한다.

<br>

## 2. TLS

서버의 보안과 별개로 서버와 클라이언트간 통신상의 암호화도 필요하다. 평문 통신의 경우 패킷 스니핑 당할 위험이 커진다.

* SSL 역시 HeartBleed, FREAK, LogJam, POODLE 등의 취약점이 있으니 SSLv3.0인 TLS를 적용한다.
* letsencrypt를 통해 무료로 TLS 인증서를 사용할 수 있다.

<br>

## 3. 무료 도메인 설정

<img width="961" alt="스크린샷 2021-09-01 오후 5 53 05" src="https://user-images.githubusercontent.com/56240505/131642356-e1f004de-a5ed-43e8-a37d-3a3a96d7388b.png">

* [내도메인](https://xn--220b31d95hq8o.xn--3e0b707e/)을 통해 무료 도메인을 발급 받는다.
* 사진과 같이 도메인에 Reverse Proxy EC2 Public IP를 입력한다.

<br>

## 4. EC2 Docker 설치 및 Nginx 컨테이너 실행

> Shell

```bash
$ sudo apt update && sudo apt install -y docker.io && sudo apt install -y docker-compose
$ sudo usermod -aG docker ubuntu
```

> Dockerfile

```bash
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf  
```

> nginx.conf

```bash
events {}

http {
  upstream app {
    server {운영 서버 EC2 private IP}:8080;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://app;
    }
  }
}
```

> Shell

```bash
$ docker build -t nextstep/reverse-proxy .
$ docker run -d -p --name -v /var/log/nginx:/var/log/nginx proxy 80:80 nextstep/reverse-proxy
```

* 이후 내 도메인에서 발급받은 주소로 브라우저 접속하면, 운영 서버에서 80번 포트로 동작 중인 웹 어플리케이션으로 연결된다.

<br>

## 5. TLS 인증서 발급

> Shell

```bash
$ docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d 'yourdomain.com' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

<img width="777" alt="스크린샷 2021-09-01 오후 6 01 27" src="https://user-images.githubusercontent.com/56240505/131720541-608b19e5-ac6f-46fd-8b63-0f10dad48306.png">

<img width="970" alt="스크린샷 2021-09-01 오후 6 02 47" src="https://user-images.githubusercontent.com/56240505/131720594-da3c7412-0414-4f64-9fcd-32d7e8c5f682.png">

* TLS 인증과 관련된 내 도메인에 적용한다.

> Shell

```bash
$ cp /etc/letsencrypt/live/[도메인주소]/fullchain.pem ./
$ cp /etc/letsencrypt/live/[도메인주소]/privkey.pem ./
```

* 인증서를 현 위치로 복사해온다.

> Dockerfile

```bash
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY fullchain.pem /etc/letsencrypt/live/[도메인주소]/fullchain.pem
COPY privkey.pem /etc/letsencrypt/live/[도메인주소]/privkey.pem
```

> nginx.conf

```bash
events {}

http {       
  upstream app {
    server {운영 서버 EC2 private IP}:8080;
  }

  # Redirect all traffic to HTTPS
  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl;  
    ssl_certificate /etc/letsencrypt/live/[도메인주소]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[도메인주소]/privkey.pem;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # 통신과정에서 사용할 암호화 알고리즘
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enable HSTS
    # client의 browser에게 http로 어떠한 것도 load 하지 말라고 규제합니다.
    # 이를 통해 http에서 https로 redirect 되는 request를 minimize 할 수 있습니다.
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL sessions
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;      

    location / {
      proxy_pass http://app;    
    }
  }
}
```

> Shell

```bash
$ docker stop proxy && rm proxy
$ docker build -t nextstep/reverse-proxy .
$ docker run -d -p 80:80 -p 443:443 -v /var/log/nginx:/var/log/nginx --name proxy nextstep/reverse-proxy
```

* 동작 중이던 컨테이너를 삭제하고 443 포트를 허용하는 컨테이너를 실행한다.

![image](https://user-images.githubusercontent.com/56240505/131724045-2a454071-2ccf-4925-9c5f-277a172777dc.png)

* HTTPS가 잘 적용되는 것을 확인할 수 있다.

<br>

## 6. 로드 밸런싱 적용

> nginx.conf

```bash
http {
  upstream backend {
    server 192.168.1.215:8080;
    server 192.168.1.208:8080;
  }

  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl;
    //SSL 관련 설정 중략

    location / {
      proxy_pass http://backend;
    }
  }
}
```

* upstream 구문에 사용할 서버들을 정의한다.
* 요청을 특정 서버 그룹으로 전달하기 위해, proxy_pass 구문을 정의한다.
* upstream 구문에 별도의 로드 밸런싱 알고리즘을 기술하지 않으면, 기본적으로 Round Robin 알고리즘이 적용된다.

<br>

## 7. 로드 밸런싱 방법 선택

Nginx Plus를 사용하면 추가적으로 선택할 수 있는 여러 방법들이 더 존재한다.

### 7.1. Round Robin

> nginx.conf

```bash
upstream backend {
   # no load balancing method is specified for Round Robin
   server backend1.example.com;
   server backend2.example.com;
}
```

* Nginx 로드 밸런싱의 기본 방법이며, 모든 요청을 각 서버로 균등하게 분배한다.
* 별도로 서버 가중치를 정의해두면 이를 고려하여 분배한다.

### 7.2. Least Connections

> nginx.conf

```bash
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

* 현재 활성화된 연결이 가장 적은 서버로 요청이 전달된다.
* 별도로 서버 가중치를 정의해두면 이를 고려하여 분배한다.

### 7.3. IP Hash

> nginx.conf

```bash
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
    server backend2.example.com down;
}
```

* Client IP 주소를 통해 요청이 어느 서버로 전달될지 결정된다.
* 해당 방법을 적용하면 동일한 주소에서 발생한 요청은 동일한 서버가 처리하는 것이 보장된다.
  * 물론 서버가 살아있는 경우에 한해서만이다.
* down 파라미터를 적용해두면 해당 서버는 로드 밸런싱 로테이션에서 일시적으로 제외된다.

### 7.4. Generic Hash

> nginx.conf

```bash
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com;
    server backend2.example.com;
}
```

* 문자열 혹은 변수 등으로 정의된 유저 키를 통해 요청이 어느 서버로 전달될지 결정된다.
* consistent 파라미터를 적용해두면, ketama consistent‑hash load balancing이 적용된다.
  * 유저 정의 해싱 키 값을 바탕으로 요청이 모든 서버로 균등하게 배분된다.

### 7.5. Random

> nginx.conf

```bash
upstream backend {
    random two least_time=last_byte;
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
    server backend4.example.com;
}
```

* 랜덤하게 선택된 서버에 요청을 전달한다.
* two 파라미터가 적용되어 있으면, Nginx는 서버 두개를 선택하고 가중치 등 및 명시된 방법 들을 고려하여 최종 서버를 선택한다.

<br>

## 8. 서버 가중치

> nginx.conf

```bash
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
}
```

* 가중치를 적용하지 않으면 기본적으로 1이 적용된다.
* 라운드로빈의 경우 요청 6개가 올 때 마다 5개는 backend1으로, 나머지 1개는 backend2로 전송된다.

<br>

---

## References

* [Nginx Docs](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
* 우아한테크코스
