---
title:  "네트워크 전체 흐름 - 웹 서버에서의 데이터 전달과 처리/ 정적,동적 라우팅" 
excerpt: "네트워크의 전체 흐름을 파악하자..."

categories:
  - Network
tags:
  - Network

date: 2022-12-03
last_modified_at: 2022-12-03

---

## 1. 웹 서버에서의 데이터 전달과 처리

![Untitled](https://user-images.githubusercontent.com/85394884/206642765-b7e9722f-26dd-4b7c-a5fa-6149e2b54b1f.png)

### 1. 물리계층 / 데이터 링크 계층 (역캡슐화)

![Untitled 1](https://user-images.githubusercontent.com/85394884/206642752-63ba2104-8cf5-4b5b-acc5-ea7dc1a65904.png)

웹 서버에 데이터가 전달

→ 데이터 링크 계층에서 이더넷 헤더의 **목적지 MAC 주소**와 **자신의 MAC 주소**를 비교 

→ 주소가 같다면 이더넷 헤더와 트레일러를 분리

→ **네트워크 계층에 전달**

### 2. 네트워크 계층 (역캡슐화)

![Untitled 2](https://user-images.githubusercontent.com/85394884/206642754-831e449f-c522-482c-9cd0-be18d3e0e1e5.png)

- IP헤더에서 목적지 IP 주소와 웹 서버의 IP 주소가 같은지 확인

    → 주소가 같다면 IP 헤더를 분리

    → **전송 계층에 전달**

### 3. 전송 계층 (역캡슐화) / 응용 계층

![Untitled 3](https://user-images.githubusercontent.com/85394884/206642757-bc8d42e6-6a7d-41a8-8dd2-4e641002b560.png)

- TCP 헤더에서 목적지 포트 번호를 확인 

    → 어떤 어플리케이션으로 전달해야 되는지 판단 후 TCP 헤더 분리

    → **응용 계층**에 전달

## 2. 정적 라우팅(****Static Routing)****

- 관리자가 라우팅 테이블에 경로를 수동으로 추가하는 방법

    → 목적지 까지의 경로를 고정하거나 목적지까지의 경로가 하나로 한정되는 경우 사용

    ![Untitled 4](https://user-images.githubusercontent.com/85394884/206642763-5672250b-da4e-4006-9e36-4d9f1d8c46df.png)

### 2-1. 장점

- 속도가 빠르다
- 설정이 비교적 간단
- 라우터 장비의 부담이 작다 → 알고리즘 사용이 없기 때문
- 특정 네트워크만 허용하므로 보안이 추가된다

### 2-2. 단점

- 네트워크 요소 연결 변경시 일일이 수동으로 변경해야한다.
- 네트워크 망이 클 경우, 사람이 직접 설정하는 것은 어렵다.

## 3. 동적 라우팅(Dynamic ****Routing)****

- 네트워크 변경을 자동으로 감지하여 라우팅 테이블을 업데이트 하거나, 라우터끼리 정보를 교환해 최적의 경로로 전환하는 기능

- RIP, OSFP와 같은 알고리즘을 사용한다.

    → 대규모 네트워크의 경우 사용

### 3-1. 장점

- 경로를 자동으로 설정해서 구성이 쉽다
- 하나의 경로가 죽으면 자동으로 다른 경로를 연결한다

### 3-2. 단점

- 다른 장비들과 통신하기 위해서 많은 대역폭을 소비한다.
- 알고리즘으로 인해 라우터 장비가 좋아야 한다.

[참고] [Static Routing vs. Dynamic Routing](https://blog.router-switch.com/2011/12/static-routing-vs-dynamic-routing/#.USBGrvLaagU)
