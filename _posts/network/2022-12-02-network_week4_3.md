---
title:  "네트워크 전체 흐름 - 스위치와 라우터에서의 데이터 전달과 처리" 
excerpt: "네트워크의 전체 흐름을 파악하자..."

categories:
  - Network
tags:
  - Network

date: 2022-12-02
last_modified_at: 2022-12-02

---

### ⭐ 이 그림을 계속 기억해둬야 이해하기 쉽다!

![Untitled](https://user-images.githubusercontent.com/85394884/206638267-b6d8353c-5c2c-4c9e-a980-55cba8de1ebe.png)

-> **끊임없는 캡슐화와 역캡슐화, 데이터계층과 네트워크계층간의 상호작용**

<br>

#### 라우터 vs 스위치

**라우터**: <span style="background-color:AntiqueWhite">'네트워크 사이'</span> 에 데이터 전송을 수행하는 기기

- 라우터에 의해 네트워크를 상호 연결할 수 있다.
- 라우터는 <span style="background-color:#daeef3">IP 주소</span>를 사용하여 네트워크 간의 데이터 전송을 수행하며, 이를 '라우팅'이라고 한다.
- 라우팅을 수행하기 위해서는 미리 라우팅 테이블에 네트워크 정보를 등록해두어야 한다.
- 라우팅 테이블에 등록되어 있지 않은 네트워크가 목적지인 데이터는 라우터에 의해 파기된다.

**스위치**: <span style="background-color:AntiqueWhite">'같은 네트워크 내부'</span>에서 데이터 전송을 수행하는 기기

- 스위치는 PC나 서버에 있어서 네트워크 입구에 해당하는 네트워크 기기이다.
- 스위치는 <span style="background-color:#daeef3">MAC 주소</span>를 사용하여 같은 네트워크의 LAN 포트 간 데이터 전송을 수행한다.
- MAC 주소는 물리적 주소라고도 한다.

✅ 스위치와 라우터 간의 차이를 쉽게 이해하려면 LAN과 WAN을 생각하면 된다. 기기는 스위치를 통해 로컬로 연결되고 네트워크는 라우터를 통해 다른 네트워크에 연결된다.

✅ IP 주소는 기기에 따라 동적으로 할당되고 변경이 가능하나, MAC 주소는 물리적 기기를 식별하는데 쓰인다. 


**데이터 전달 흐름**

💻 스위치 A → 라우터 A → 라우터 B → 스위치B 

<br>

### 1️. 스위치 A

![Untitled 1](https://user-images.githubusercontent.com/85394884/206638258-5d6f4edd-55be-4c75-9066-b2fa742e190f.png)

- 그저 전달
- 컴퓨터에서 보낸 데이터 받아서 전기신호로 변환 - 물리계층
- 라우터A에 전송한다 - 데이터 링크 계층

![Untitled 2](https://user-images.githubusercontent.com/85394884/206638261-e7a7e1cd-82da-45e4-8449-26e4e1b635dd.png)

- 데이터 상태 - 이더넷프레임

### 2️. 라우터 A

![Untitled 3](https://user-images.githubusercontent.com/85394884/206638263-d7460358-5d29-44fa-968f-89c74ddb34a7.png)

1. **<span style="background-color:AntiqueWhite">MAC 주소 확인, 역캡슐화</span>** - **데이터 링크 계층 1차**
    - 받은 데이터의 목적지 MAC 주소와 자신의 MAC주소 비교 
    - 같으면 이더넷 헤더와 트레일러 분리 
    
2. **<span style="background-color:AntiqueWhite">목적지 IP 주소 비교</span>** - **네트워크 계층**
    - 라우터 A의 라우팅 테이블에서 목적지 IP 주소의 경로를 알아야 라우팅 가능 
    
        - 출발지 IP 주소를 라우터 외부 IP 주소(공인ip)로 변경한다.
        
            라우터에는 두개의  IP 주소가 있다. 
            
            - 로컬 네트워크 자체 주소
            - 인터넷의 외부 네트워크와 통신하는 데 사용되는 외부 공용 IP 주소 (공인 ip)
            
            우리는 가정에서 유무선공유기를 통해 라우터를 거쳐 인터넷망에 접속한다. 
            
            이때 내 컴퓨터는 공유기로부터 ip주소를 할당받아 인터넷망에 접속하게 되는데, 공유기로부터 할당받은 이 ip 주소가 `내부ip 주소` 다.
            
            외부 네크워크에서는 방화벽 때문에 이 내부ip로 직접 접근할 수 없다.  
        
3. **<span style="background-color:AntiqueWhite">캡슐화</span>** - **데이터 링크 계층 2차**
    - 라우터B에 보내질 수 있게 캡슐화한다. 
    
        ![Untitled 4](https://user-images.githubusercontent.com/85394884/206638264-a195c765-ba9c-4fe9-88dc-6157b24dd984.png)
    
4. **<span style="background-color:AntiqueWhite">전송</span>** - **물리계층**

### 3️. 라우터 B

- 라우터 A가 하는 일과 동일

- 다만 2-2 단계에서 출발지 IP 주소를 바꾸어 주었던 것을 다시 라우터 B의 내부 IP 주소로 변경하는 과정만 다름 

    ![Untitled 5](https://user-images.githubusercontent.com/85394884/206638266-329e6b27-47ca-4226-99a5-a01a3a1e875a.png)

### 4️. 스위치 B

- 라우터 B가 보내 준 위와 같은 데이터를 웹 서버에 전송한다.
