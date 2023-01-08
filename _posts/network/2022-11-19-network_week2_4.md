---
title:  "OSI 물리계층" 
excerpt: "OSI 물리계층"

categories:
  - Network
tags:
  - Network

date: 2022-11-19
last_modified_at: 2022-11-19

---

# 물리 계층의 역할과 랜 카드의 구조

## 1. 전기 신호란?

전기를 통해 전달하는 신호.

**아날로그신호**와 **디지털 신호**가 있다.

### 1-1. 아날로그 신호

전류, 전압이 연속적으로 변하는 연속신호.

주로 전화 회선, 라디오 방송에 사용된다.

![Untitled](https://user-images.githubusercontent.com/85394884/206538604-5e981526-f700-4a85-baca-f500b9e91fd7.png)

### 1-2. 디지털 신호

0과 1만 존재하는 비 연속 신호(이산 신호)

![Untitled 1](https://user-images.githubusercontent.com/85394884/206538574-2390982f-1ab8-4315-ad52-6c70fe5b165c.png)

### 1-3. 물리계층이란

![Untitled 2](https://user-images.githubusercontent.com/85394884/206538575-d1f505b8-47c0-4dac-8b97-b0be761dd526.png)

전기 신호를 전달하기 위한 물리적 계층.

컴퓨터는 0과 1로 이루어진 비트열을 사용하기 때문에, 이를 전기신호로 바꾸어주기 위한 기술

## 2. 랜 카드란?

![Untitled 3](https://user-images.githubusercontent.com/85394884/206538582-6b5b403e-dc6a-43de-a83f-6d10b87ccafd.png)

- 컴퓨터와 전송 매체를 연결하는 장치
- 컴퓨터와 네트워크 사이에서 인터페이스 역할을 하며 프로토콜의 처리를 담당
- 컴퓨터의 디지털 데이터를 전기신호로 변환
- 전기신호를 디지털 데이터로 변환
- NIC(Network Interface Card)라고 부른다.

<br>

# 케이블의 종류와 구조

## 1. 트위스트 페어 케이블이란?

UTP 케이블, STP 케이블이 있다.

![Untitled 4](https://user-images.githubusercontent.com/85394884/206538584-3e5e8566-1abf-4253-bcfe-6725b882276f.png)

### 1-1. UTP(Unshielded Twist Pair) 케이블

실드로 보호되어 있지 않은 케이블.

노이즈의 영향을 받기 쉽지만, 저렴하기 때문에 일반적으로 많이 사용한다.

### 1-2. STP(Shielded Twist Pair) 케이블

실드로 보호한 케이블

노이즈의 영향을 적게 받지만, 비싸다.

### 1-3. 노이즈의 영향

![Untitled 5](https://user-images.githubusercontent.com/85394884/206538586-7922dcf9-f09f-4c90-bb28-b6b6c21e5405.png)

노이즈를 받으면 신호의 형태가 왜곡된다.

### 1-4. UTP 케이블의 분류

![Untitled 6](https://user-images.githubusercontent.com/85394884/206538589-d87e0865-c6dd-4687-bb30-5e47f5f444a7.png)

데이터 전송 품질에 따라 UTP 케이블을 분류한다.

### 1-5. 랜 케이블(LAN Cable)

UTP 케이블은 일반적으로 랜 케이블이라 부른다.

## 2. 다이렉트 케이블과 크로스 케이블이란?

### 2-1. 다이렉트 케이블

![Untitled 7](https://user-images.githubusercontent.com/85394884/206538592-c58a1380-8c79-43ce-89cb-9884ad9937ef.png)

- 케이블의 배선 순서가 같은 케이블
- 일반적으로 사용되는 케이블

### 2-2. 크로스 케이블

![Untitled 8](https://user-images.githubusercontent.com/85394884/206538594-18429a9c-e43a-4d45-b2a4-a18fad52dab6.png)

- 케이블 배선 순서가 엇갈려 있는 케이블
- 서로 다른 계층간의 장비끼리 통신할 때 사용. (Hub to Hub, PC to PC)

    > 1번 → 3번 <br>2번 → 6번 <br> 위와 같이 연결된다.

### 2-3. 사용하지 않는 선

다이렉트 케이블, 크로스 케이블 모두 4, 5, 7, 8번 선을 사용하지 않는다.

다이렉트 케이블 : 컴퓨터와 스위치를 연결할 때 사용

크로스 케이블 : 컴퓨터  간에 직접 랜 케이블로 연결할 때 사용

<br>

# 리피터와 허브의 구조

## 1. 리피터란?

![Untitled 9](https://user-images.githubusercontent.com/85394884/206538597-dc428e94-2f12-4831-8805-16c8c6de7277.png)

- 신호를 수신하여 증폭시키는 장치. (물리계층)
- 일대일 통신을 지원
- 요즘은 허브 등 다른 네트워크 장비가 리피터의 기능을 지원한다.

## 2. 허브란?

![Untitled 10](https://user-images.githubusercontent.com/85394884/206538600-63055cbb-c68a-4434-bf26-7566ffbcf06d.png)

- 랜을 구성할 때 가까운 거리에 있는 장비들을 케이블 사용하여 연결하는 장치
- 여러개의 포트를 갖고 있어, 다수의 PC와 통신하며 네트워크를 구성
- 리피터와 멀티포트의 역할 제공
- 허브는 스스로 판단하지 않기에 **더미 허브** 라고 부른다.
- 받은 데이터를 **연결된 모든 포트에 전송**
    
    → 단점을 극복할 수 있는 스위치(스위칭 허브)를 사용

<br>

---

## References

* [쉽게 이해하는 네트워크 10. TCP/IP 네트워크 인터페이스 계층의 역할과 데이터 전송 (ft. 랜카드와 MAC 주소)](https://better-together.tistory.com/101)
* [UTP 케이블의 정의와 종류](https://ghdwn0217.tistory.com/74)
* [[Network] 리피터, 허브, 스위치, 라우터](https://minkwon4.tistory.com/288)