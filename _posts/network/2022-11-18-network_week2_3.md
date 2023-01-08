---
title:  "캡슐화와 역캡슐화" 
excerpt: "캡슐화와 역캡슐화"

categories:
  - Network
tags:
  - Network

date: 2022-11-18
last_modified_at: 2022-11-18

---

# 캡슐화 & 역캡슐화(Encapsulation & Decapsulation)

- 네트워크를 통해 데이터를 보낼 때는 캡슐화와 역캡슐화의 과정이 이루어진다.
- 정확히는 데이터 **<span style="background-color:AntiqueWhite">전송</span>** 시 캡슐화, **<span style="background-color:AntiqueWhite">수신</span>** 시 역캡슐화가 이루어진다.
- 송신 측에서 데이터에 헤더를 붙이고 하위 계층에 보내는 것을 **<span style="background-color:AntiqueWhite">'캡슐화'</span>** 라고 하며, 헤더에는 데이터를 전달 받을 상대방에 대한 정보가 포함되어 있다.
- 반대로, 수신 측에서 헤더를 제거하고 상위 계층에 보내어 최초로 보낸 데이터 형태로 받는 것을 **<span style="background-color:AntiqueWhite">'역캡슐화'</span>** 라고 한다.

**<span style="background-color:MistyRose">그렇다면 헤더는 왜 필요한가?</span>**

컴퓨터 공학에서는 컴퓨터에게 가까운 부분일 수록 ‘**낮다**’ 혹은 ‘**뒤에 있다**’ 라는 표현을, 사람에게 가까운 부분일 수록 ‘**높다**’, ‘**앞에 있다**’라는 표현을 사용하는데, OSI 7계층에서도 마찬가지로 적용된다.

즉, 낮은 계층(1계층)일수록 기계에 가깝고, 높은 계층(7계층)일수록 사람에게 가까운 부분이라는 것이다.

그렇기 때문에, 하위 계층에 속한 프로토콜들은 상대적으로 개발자가 접할 일이 적으며 보통 OS에서 알아서 처리해주기 때문에 상세 동작까지 신경 쓸 필요는 없다.

하지만, 그렇다고 하위 계층에서 일어나는 일을 전혀 모른다는 것은 다른 얘기이다. 오류가 생겼을 때 전혀 손을 못 대는 경우가 생길 수 있다.

그렇기 때문에, 각 계층에서 그 계층이 사용하는 프로토콜의 대략적인 작동 원리나 개요 정보에 대한 이해 정도의 수준까진 필요하며 우리는 이것을 각 계층에서 **<span style="background-color:MistyRose">'헤더'</span>** 에 담는다고 생각하면 편할 것이다.

💡 단, ‘헤더’는 헤더가 붙은 그 계층에서만 의미를 가진다. 그렇기 때문에, 캡슐화를 거친 데이터는 하위 계층에서는 알 수 없고, 반대의 경우도 역캡슐화를 거치며 하위 계층의 헤더가 떼어진 상태로 전달 받는다.

## 1. 캡슐화

![Untitled](https://user-images.githubusercontent.com/85394884/206528880-e262c277-8386-4b98-9d31-6973df8ddba6.png)


- 일반적으로 5,6,7계층은 합쳐서 7계층의 응용 계층에 포함해서 생각하기 때문에 응용 계층으로  표현했다.

### <span style="background-color:AntiqueWhite">1-1. 과정</span>

1. 송신 측의 컴퓨터에서 웹 사이트에 접속 시 **응용 계층**에선 웹사이트 접속하기 위한 데이터 생성
2. 해당 데이터는 하위 계층인 **전송 계층**에 전달되고, 전송 계층에서 신뢰할 수 있는 통신이 이루어지도록 응용 계층에서 만들어진 데이터에 <span style="background-color:MistyRose">헤더</span>를 붙임
3. 전송 계층에서의 데이터를 다른 네트워크와 통신하기 위해 **네트워크 계층**에서 <span style="background-color:MistyRose">헤더</span>를 붙임
4. 네트워크 계층에서 만들어진 데이터에 물리적인 통신 채널을 연결하기 위해 **데이터 링크 계층**에서 <span style="background-color:MistyRose">헤더와 트레일러</span>를 붙인다.
    
    💡 **트레일러란?**
    
    - 데이터 전달 시 데이터의 마지막에 추가하는 정보
    
5. 최종적으로, **물리 계층**에선 <span style="background-color:MistyRose">전기 신호</span>로 변환돼서 수신측에 도착한다.

<br>

## 2. 역캡슐화

### <span style="background-color:AntiqueWhite">2-1. 과정</span>

1. 전기 신호로 수신측에 도착하면 수신측 각 계층에서 송신 측 계층에서 생성한 헤더를 제거하면서 상위 계층에 데이터를 전달한다.  
2. 송신 측 데이터 링크 계층에서 추가된 <span style="background-color:MistyRose">헤더와 트레일러</span>를 수신 측 데이터 링크 계층에서 제거
3. 네트워크 계층 <span style="background-color:MistyRose">헤더</span>를 제거
4. 모든 헤더가 제거된 <span style="background-color:MistyRose">최초의 데이터 상태</span>로 수신측에 도착

<br>

## 3. VPN

### <span style="background-color:AntiqueWhite">3-1. 개념</span>

- 가상 사설망, 가상의 사설 네트워크
- 쉽게 설명하면 우리가 사용하는 인터넷 망을 통해서 내 전용망처럼 사용하는 기술을 말함
    - 인터넷 망은 보안에 취약한데, 이를 이용하여 중요한 자료 등을 기업 본사에서 가져와야 한다면 위험하다.
    - 그래서 VPN으로 접속하면 기업 본사와 내 PC사이의 **<span style="background-color:MistyRose">암호화된 터널</span>** 이 만들어져 외부에선 접근이 불가능하게 한다.
    
        ‼️ 즉, 특정 네트워크와 마치 유선으로 연결된 것 같은 느낌
    
### <span style="background-color:AntiqueWhite">3-2. 방식</span>

1. `IPSEC`  방식 (3계층 네트워크 계층) (IP 프로토콜 사용)
    - 하드웨어 VS 하드웨어 방식
    - 기업 본사와 지사를 연결할 때 **<span style="background-color:MistyRose">VPN 하드웨어 장비</span>** 를 통해서 연결하는 방식
    - 본사와 지사처럼 고정되어 있는 장소 연결에 사용

    ![Untitled 1](https://user-images.githubusercontent.com/85394884/206528835-3f2ad048-ec21-4e5c-9779-815124a010ba.png)

2. `SSL` VPN (6계층 표면 계층) (TCP 프로토콜 사용)
    - 하드웨어 VS 소프트웨어 방식
    - 지사가 아닌 개인의 PC의 경우엔 모든 개인이 VPN 하드웨어 장비를 가지고 다닐 수 없기에 **<span style="background-color:MistyRose">웹 브라우저</span>** 를 통해 **<span style="background-color:MistyRose">클라이언트 소프트웨어</span>** 를 설치하여 연결하는 방식
    - 인터넷을 통해 기업 본사 하드웨어에 접속하면 자동으로 소프트웨어 다운로드 후에, 본사 VPN과 연결됨
    
    ![Untitled 2](https://user-images.githubusercontent.com/85394884/206528847-ed85b8f4-f186-4416-83e3-a0e4496a6b41.png)
    

### <span style="background-color:AntiqueWhite">3-3. 심화</span>

‼️ `IPSEC`  방식과 `SSL` VPN이 각각 3계층과 6계층에서 동작한다는 의미는 무엇인가?

1. `IPSEC` VPN 터널을 통해 이동하는 패킷은 패킷이 가진 <span style="background-color:MistyRose">헤더, IP주소, 데이터 부분</span>이 모두 암호화 처리되고, 대신 IPSEC VPN에서 추가한 헤더와 IP주소로 데이터가 전달된다는 의미
    
    ![Untitled 3](https://user-images.githubusercontent.com/85394884/206528860-a5cbd893-c8cd-4be1-b677-9346a7ba8a40.png)
    
2. `SSL` VPN 터널을 통해 이동하는 패킷은 <span style="background-color:MistyRose">데이터</span> 부분만 암호화 처리된다는 의미
    
    ![Untitled 4](https://user-images.githubusercontent.com/85394884/206528872-63626f40-2624-4da8-9e93-bfebcc8f7e57.png)