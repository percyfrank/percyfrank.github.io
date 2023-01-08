---
title:  "OSI 데이터 링크 계층 - 스위치와 허브, 이더넷의 종류와 특징" 
excerpt: "데이터 링크 계층을 알아보자."

categories:
  - Network
tags:
  - Network
 
date: 2022-11-22
last_modified_at: 2022-11-22

---

- 0,1로 이루어진 비트열 데이터를  주고 받기 위해 첫번째 계층인 물리계층 (LAN)에서 전기신호로 변환/해석 하는 기능을 했다. 

- 랜에서 데이터를 주고 받기 위한 기술을 가진 두 번째 계층인 데이터 링크 계층에 대해 알아본다. 

- **데이터 링크계층** - 네트워크 장비 간(LAN)에 신호를 주고 받는 규칙  ex) `이더넷`
    - 데이터 충돌 방지 - `CSMA/CD (레거시)` , `스위치`
    - 더미허브에서 문제점이였던 불필요한 곳까지도 전송을 제어할 수 있음
- `MAC 주소` - 랜 카드를 제조할 때 정해지는 물리적인 주소
    - 랜카드 별로 정해진 고유번호
    - MAC주소를 통해 데이터 링크계층에서의 `헤더`를 구성함.

<br>

- 스위치의 구조
    - MAC 주소 테이블 관리 - ARP
- 허브 vs 스위치
    - 충돌 관리 , 충돌 도메인 범위, 전이중/반이중 통신
- 이더넷의 종류와 특징 
    - 케이블 종류, 통신 속도에 따른 규격

<br>

## 1. 스위치의 구조

**스위치**는 **데이터 링크 계층**에서 작동하며, `레이어 2 스위치` 또는 `스위칭 허브` 라고도 불린다.

이전에는 허브로 많이 쓰였던 기능인데 충돌방지면에서 스위치가 더 뛰어나서 요즘 랜 허브라고 불리는 것들은 이 스위치를 사용하는 경우가 많다.  

- 우리가 흔히 아는 **허브**는 이더넷허브라고 해서 스위치가 아닌걸 사용하고 있었고 (랜 케이블 꽂는거 하나 인 허브), **랜 허브**는 랜 케이블 여러개 꽂을 수 있어 충돌방지를 위해 스위치를 다 쓰고 있었다. 

[허브] <https://www.coupang.com/vp/products/5122557162?itemId=7003577587&vendorItemId=74295845366&src=1042503&spec=10304991&addtag=400&ctag=5122557162&lptag=10304991I7003577587&itime=20221122154609&pageType=PRODUCT&pageValue=5122557162&wPcid=16690995691816866612848&wRef=&wTime=20221122154609&redirect=landing&gclid=Cj0KCQiA4OybBhCzARIsAIcfn9lZ0wA0AgkRbrOLHNcuV2fOG_StTQ0t-AMtYz1i5wDYlSCu7lLyxkoaAjdHEALw_wcB&campaignid=18860239423&adgroupid=&isAddedCart=>

[랜 허브] <https://www.coupang.com/np/search?component=&q=%EB%9E%9C+%ED%97%88%EB%B8%8C&channel=user>

**스위치 내부**에는 `MAC 주소 테이블`이 있다

### 1-1. MAC 주소 테이블

스위치의 포트 번호와 해당 포트에 연결되어 있는 컴퓨터의 MAC 주소가 등록되는 데이터베이스이다.

- **MAC 주소 학습 기능** 
   
   처음 스위치의 주소 테이블에는 아무것도 등록되어 있지 않다.

   컴퓨터에서 목적지 MAC 주소가 추가된 `프레임`이라는 데이터가 전송되면 MAC 주소 테이블을 확인하고, 
   출발지 MAC 주소가 등록되어 있지 않으면 MAC 주소를 포트와 함께 등록한다. 
   
   -> 허브에는 없는 기능

- **플러딩**

    ![Untitled](https://user-images.githubusercontent.com/85394884/206549498-3d46ea09-9590-4dac-9365-a19ed64103af.png)

- **필터링**

    ![Untitled 1](https://user-images.githubusercontent.com/85394884/206549474-f8f7caf1-d572-497e-b8cf-03746b56131a.png)

<br>

## 2. 데이터가 케이블에서 충돌하지 않는 구조

### 2-1. 통신 방식에 따른 충돌 관리
- 통신 방식에는 **전이중 통신 방식** 과 **반이중 통신 방식**이 있다. 
   
- **전이중** 은 데이터를 동시에 전송해도 충돌이 발생하지 않지만, **반이중**은 발생한다. 
  
  ![Untitled 2](https://user-images.githubusercontent.com/85394884/206549483-66fb5147-6886-4293-87d3-627be8cfe67e.png)
  
  **충돌 도메인** - 충돌이 발생할 때 그 영향이 미치는 범위  
  
  예를 들어 허브는 연결되어 있는 컴퓨터 전체가 하나의 **충돌 도메인**이 된다. 
  
  **충돌 도메인**의 범위가 좁아야 네트워크 지연이 적어져 통신 효율이 높아진다. 
    
### 2-2. 허브 vs 스위치
- 허브 - 반이중 통신 방식, 스위치- 전이중 통신 방식
- 충돌 도메인의 범위가 허브가 넓어 통신 효율이 떨어진다.

<br>

## 3. 이더넷의 종류와 특징

이더넷은 네트워크 장비 간(LAN)에 신호를 주고 받는 규칙이며, OSI 모델의 **물리 계층**
(LAN카드, 케이블 영역)에서 신호와 배선, **데이터 링크 계층**에서 MAC 주소와 프로토콜의 형식을 정의한다.

- LAN 케이블 종류나 통신 속도에 따라 다양한 규격으로 분류된다.
    
    ![Untitled 3](https://user-images.githubusercontent.com/85394884/206549487-335608b0-2971-4d3c-9280-f53b94f88623.png)
    
    현재 일반적인 포트는 `1000BASE-T`
    
- 규격 이름의 의미
    
    ![Untitled 4](https://user-images.githubusercontent.com/85394884/206549496-fa8c996b-fa60-44d7-a331-cfcdeb201c87.png)
    
    이때 전송 방식 뒤에 케이블 명이 아니라 숫자로 되어 있는 경우는, 케이블의 최대길이를 나타낸것이며 100m 단위로 표현된다.
    
<br>

## 4. 추가 검색 사항

### ARP(주소 결정 프로토콜)이란?

[참고] <https://coding-factory.tistory.com/720>


**주소 결정 프로토콜(Address Resolution Protocol, ARP)은 네트워크 상에서 IP 주소를 MAC 주소로 대응시키기 위해 사용된다.**

- 필요한 이유

  처음 통신을 시작할 때는 상대방의 MAC Address를 모르기 때문에 그 때 사용하는 프로토콜이다. 
  
  IP주소만 알면 데이터를 전송할 수 있다고 생각하지만 사실 IP주소는 인터넷에 연결되어 있는 호스트나 라우터 장비의 인터페이스에 할당된 주소이다. 
  
  PC 주소가 아닌 **네트워크에 할당된 주소**인것. 
  
  실생활로 예시를 들자면, 우체국 택배에 “서울특별시 동작구 A빌딩” 까지만 적혀있고 몇호인지 상세정보는 적혀있지 않는 것과 같다.
  
  그래서 ARP 프로토콜을 활용하여 MAC 주소를 알아내서 정확한 위치를 찾아내야 하는 것이다. 

  그래서 위에서 스위치를 알아봤을 때, MAC 주소가 테이블에 저장되지 않으면 스위치에 모든 연결된 컴퓨터에 데이터가 전송되는 것이다 -> 플러딩 ( A빌딩 모두에게 전송 ) 

### 이더넷, 인터넷, 웹의 차이

[참고] <https://bentist.tistory.com/33>