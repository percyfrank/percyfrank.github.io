---
title:  "네트워크 전체 흐름 - 랜 카드에서의 데이터 전달과 처리" 
excerpt: "네트워크의 전체 흐름을 파악하자."

categories:
  - Network
tags:
  - Network

date: 2022-12-01
last_modified_at: 2022-12-01

---

## 1. 네트워크의 구성

**OSI 모델의 상위 응용 계층부터 하위 물리 계층까지에서 어떤 일이 일어나는지?**

![1](https://user-images.githubusercontent.com/85394884/206635616-4e6a0626-df49-46fb-b9c0-646fd88133e1.png)

**물리 계층**

- 데이터를 전기 신호로 변환하는 데 필요하다.

**데이터 링크 계층**

- 랜에서 데이터를 송수신하는 데 필요하다.

**네트워크 계층**

- 다른 네트워크에 있는 목적지에 데이터를 전달하는 데 필요하다.

**전송 계층**

- 목적지에 데이터를 정확하게 전달하는 데 필요하다.

**응용 계층**

- 애플리케이션 등에서 사용하는 데이터를 송수신하는 데 필요하다.

<br>

## 2. 컴퓨터의 데이터가 전기 신호로 변환되는 과정

### 컴퓨터 → 응용(7) 계층으로 전달 (캡슐화)

**<span style="background-color:AntiqueWhite">응용 계층</span>**

1. 3-Way-Handshake 수행 
2. HTTP 프로토콜로 GET으로 보내기

![2](https://user-images.githubusercontent.com/85394884/206635622-0e017239-c071-4902-a6e5-4e2c825aa723.png)

**3 Way-Handshake란?** 
- 전송제어 프로토콜(TCP)에서 통신을 하는 장치간 서로 연결이 잘 되어있는지 확인하는 과정/방식
- **송수신자**(데이터를 주고 받는 두 사람이라고 생각하면...) **사이에 연결을 확인하는 과정**

![3way](https://user-images.githubusercontent.com/85394884/206635624-e61d0ea8-f9cf-4a7e-9520-60a3c248f713.png)

- 1단계 : 들려? → 2단계 : 응 들려! 너도 들려? → 3단계 : 응 들려!

**HTTP 메시지가 전송 계층에 전달**

**<span style="background-color:AntiqueWhite">전송 계층</span>** - TCP 헤더가 붙는다

- TCP 헤더에서 출발지 포트 번호, 목적지 포트 번호
- 출발지 포트 번호 - 잘 알려진 포트가 아닌 1025번 이상인 포트 중에서 무작위로 선택

⭐TCP 헤더를 가진 데이터 - **세그먼트**

![4](https://user-images.githubusercontent.com/85394884/206635626-57a01e37-0e85-42e4-9927-4a90790b1dd4.png)

**<span style="background-color:AntiqueWhite">네트워크 계층</span>** - IP 헤더가 붙는다

- 네트워크 계층에서 전달받은 **세그먼트**에 IP 헤더를 붙인다.
- IP 헤더에 출발지 IP 주소와 목적지 IP 주소가 추가된다.

![5](https://user-images.githubusercontent.com/85394884/206635630-f28070db-3605-4357-a69b-c6a58ce029f4.png)

**<span style="background-color:AntiqueWhite">데이터 링크 계층</span>** - 이더넷 헤더, 트레일러 추가

- 이더넷 헤더가 있는 데이터 - 이더넷 프레임

**<span style="background-color:AntiqueWhite">물리 계층</span>**

- 전기 신호로 변환되어 네트워크로 전송

![6](https://user-images.githubusercontent.com/85394884/206635633-4738ce0e-261b-4ff0-93d2-c7cc69e986c4.png)