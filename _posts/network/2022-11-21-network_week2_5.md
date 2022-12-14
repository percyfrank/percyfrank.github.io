---
title:  "OSI 데이터 링크 계층 - 데이터 링크 계층의 역할과 MAC 주소, 허브" 
excerpt: "데이터 링크 계층을 알아보자."

categories:
  - Network
tags:
  - Network
  
date: 2022-11-21
last_modified_at: 2022-11-21

---

## 1. 데이터 링크 계층의 역할과 이더넷

### 1-1. 이더넷이란?

> 데이터 링크 계층은 네트워크 장비 간에 신호를 주고받는 규칙을 정하는 계층으로, 랜에서 데이터를 정상적으로 주고받기 위해 필요한 계층이다.
일반적으로 많이 가장 많이 사용되는 규칙
> 

- 랜에서 적용되는 규칙
- 허브와 같은 장비에 연결된 컴퓨터와 데이터를 주고받을 때 사용한다.

### 1-2. 허브

- 약해지거나 파형이 뭉그러진 전기 신호를 복원시키고, 해당 전기 신호를 전달받은 포트를 제외한 나머지 포트에 전달한다.
- 같은 허브를 사용하는 랜 환경에서는 특정한 컴퓨터 한 대에 데이터를 보내려고 해도 다른 모든 컴퓨터에 전기 신호가 전달된다.

    ![1](https://user-images.githubusercontent.com/85394884/206546413-5e273b36-18f9-48eb-8061-ce580cce7cce.png)

📢 **<span style="color:indianred">특정 컴퓨터에 데이터를 보내는데, 관계없는 다른 컴퓨터들이 데이터의 내용을 못 보게 하는법?</span>**

- 보내려는 데이터에 목적지 정보를 추가해서 보내면 목적지 이외의 컴퓨터는 데이터를 받더라도 무시하게 되어있다.

    ![2](https://user-images.githubusercontent.com/85394884/206546420-bdca6873-d705-41f2-866a-3ecbb65df757.png)

### 1-3. 충돌

- 컴퓨터 여러 대가 동시에 데이터를 보내면 데이터들이 서로 부딪힐 수 있다(충돌이 일어날 수 있다)

    ![3](https://user-images.githubusercontent.com/85394884/206546425-5e3e4f16-95c1-41c3-af3a-b17d503ae8ff.png)

### 1-4. CSMA/CD

- 데이터가 동시에 케이블을 지나가면 충돌할 수 밖에 없어서 데이터를 보내는 시점을 늦추게 되는데, 이더넷에서 시점을 늦추는 방법을 CSMA/CD 라고 한다
- CSMA/CD: Carrier Sense Multiple Access with Collision Detection의 약어로 반송파 감지 다중 접속 및 충돌 탐지라는 뜻을 가진다.

    > CS: 데이터를 보내려고 하는 컴퓨터가 케이블에 신호가 흐르고 있는지 아닌지 확인한다는 규칙
    <br>MA: 케이블에 데이터가 흐르고 있지 않다면 데이터를 보내도 좋다는 규칙
    <br>CD: 충돌이 발생하고 있는지를 확인한다는 규칙
    > 

    ![4](https://user-images.githubusercontent.com/85394884/206546430-2259cfb4-0310-46ba-b499-f5e99d05f85f.png)


📢 하지만! 지금은 효율이 좋지 않다는 이유로 **스위치(switch)**라는 네트워크 장비를 사용하여 충돌을 막는다.

<br>

## 2. MAC 주소의 구조

### 2-1. MAC 주소란?

- 랜 카드는 비트열(0과 1)을 전기 신호로 변환하는데, 이런 랜카드에는 MAC(Media Access Control Address) 주소라는 번호가 정해져 있다.
- 제조할 때 새겨지기 때문에 물리 주소라고도 부른다.
- 전 세계에서 유일한 번호로 할당되어 있다.
- 중복되지 않도록 규칙이 명확하게 정해져 있고 48비트 숫자로 구성되어 있다.
  
  (앞쪽 24비트는 랜카드를 만드는 제조사 번호 )
  
  (뒤쪽 24비트는 제조사가 랜카드에 붙인 일련번호)

    ![5](https://user-images.githubusercontent.com/85394884/206546433-836bdd0d-06cb-487a-a6fa-20152df61c0c.png)

### 2-2. MAC 주소를 사용한 통신

- OSI 모델이나 TCP/IP 모델에서는 각 계층에 헤더를 붙이는데, OSI 모델에서는 데이터 링크 계층에 해당하고 TCP/IP 모델에서는 네트워크 계층에 해당한다.
- 이 계층에서는 **이더넷 헤더**와 **트레일러**를 붙인다.

    **이더넷 헤더**

    ![6](https://user-images.githubusercontent.com/85394884/206546436-24035ad2-fd06-4f24-83a4-ed869743e56a.png)

    - 이더넷 유형은 이더넷으로 전송되는 상위 계층 프로토콜의 종류를 나타낸다.
    - 유형에 프로토콜 종류를 식별하는 번호가 들어간다는 내용 정도만 기억.

        | 유형 번호 | 프로토콜 |
        | --- | --- |
        | 0800 | IPv4 |
        | 0806 | ARP |
        | 8035 | RARP |
        | 814C | SNMP over Ethernet |
        | 86DD | IPv6 |

    **트레일러**

    - 이더넷 헤더 외에, 데이터 뒤에 추가한다.
    - FCS(Frame Check Sequence)라고도 하는데, 데이터 전송 도중에 오류가 발생하는지 확인하는 용도로 사용한다.

        ![7](https://user-images.githubusercontent.com/85394884/206546440-aa8f5f6c-f0d1-4dba-afc2-052c7ef250dd.png)

### 2-3. 허브를 통해 다른 컴퓨터로 데이터를 보내는 과정

- 컴퓨터 1은 이더넷 헤더에 데이터의 목적지인 컴퓨터3 의 MAC주소(목적지 MAC 주소)와 자신의 주소(출발지 MAC 주소)를 넣고 데이터를 전송 - **이 때 컴퓨터 1에서 캡슐화 발생**
- 데이터 링크 계층에서 데이터에 이더넷 헤더와 트레일러를 추가하여 **프레임**을 만들고, 물리 계층에서 이 프레임 비트열을 **전기 신호**로 변환하여 네트워크를 통해 전송

    ![8](https://user-images.githubusercontent.com/85394884/206546442-0c8ec2c2-b4e6-474c-a624-48ffb2ed5684.png)

- 컴퓨터 3의 물리 계층에서 전기 신호로 전송된 데이터를 비트열로 변환하고, 
데이터 링크 계층에서 이더넷 헤더와 트레일러 분리 (역캡슐화)
(컴퓨터 3은 역캡슐화 한 다음에 데이터 수신)

📢 만약 컴퓨터 1과 2가 동시에 컴퓨터 3에 데이터를 전송하는 경우? → **CSMA/CD** 방식이 사용된다.

![9](https://user-images.githubusercontent.com/85394884/206546447-9e26862f-1e6f-4ead-a6ad-8ba103ce3c58.png)