---
title:  "OSI 응용 계층 - DNS 서버, 메일 서버(SMTP & POP3)" 
excerpt: "응용 계층에 대해 알아보자."

categories:
  - Network
tags:
  - Network
 
date: 2022-11-30
last_modified_at: 2022-11-30

---

# DNS 서버의 구조

## 1. 개요

- 인터넷 호스트의 식별자
    - 호스트 이름 : www.facebook.com, www.google.com
    - IP 주소 : 121.7.106.83

- **<span style="background-color:MistyRose">호스트 이름</span>** 은 사용자 입장에서 기억하기는 쉽지만 해당 호스트의 위치 정보를 제공하지 않고, 숫자가 아닌 문자로 구성되므로 라우터가 처리하기 어렵다.

- **<span style="background-color:MistyRose">IP 주소</span>** 는 계층구조로 주소를 왼쪽에서부터 오른쪽으로 조사하면 해당 호스트의 위치 정보를 상세히 알 수 있으나, 사람이 기억하기 쉽진 않다.

- 이러한 선호 차이 절충을 위해, **<span style="background-color:MistyRose">호스트 이름을 IP 주소로 변환해주는 서비스</span>** 를 **<span style="background-color:MistyRose">DNS</span>** 라고 한다.

❓ 브라우저에서 [www.someschool.edu/index.html](http://www.someschool.edu/index.htmldmf) 을 요청 상황

- 사용자의 호스트는 HTTP 요청 메시지를 웹 서버 www.someschool.edu로 보낼 수 있도록 IP 주소를 알아야 한다.
    1. 브라우저는 URL로부터 호스트 이름 www.someschool.edu를 추출하고 이를 DNS 애플리케이션 클라이언트 측에 넘긴다.
    2. DNS 클라이언트는 DNS 서버로 호스트 이름을 포함한 질의를 보낸다.
    3. DNS 클라이언트는 호스트 이름에 대한 IP 주소를 가진 응답을 받는다.
    4. 브라우저가 DNS로부터 IP 주소를 받으면, 브라우저는 해당 IP 주소와 그 주소의 80번 포트에 위치하는 HTTP 서버 프로세스로 TCP 연결을 초기화한다.

- DNS의 추가 서비스
    - 호스트 에일리어싱(host aliasing) : 별칭 호스트 이름에 대한 정식 호스트 이름을 얻기 위해 이용
    - 메일 서버 에일리어싱(mail server aliasing)
    - 부하 분산

## 2. DNS 동작 원리

- 전 세계에 분산된 많은 DNS 서버뿐만 아니라 DNS 서버와 질의를 하는 호스트 사이에서의 애플리케이션 계층 프로토콜로 구성되어 있다.

❓ 만약 중앙 집중 방식의 하나의 DNS 서버로 모든 질의를 응답하는 방식이라면 어떤 문제가 생길 것인가?

- 서버의 고장 : 전체 인터넷 작동 불가
- 트랙픽 양 : 모든 DNS 질의 처리
- 먼 거리로부터의 심각한 지연
- 유지관리 : 확장성 불가

✅ 그래서 DNS는 분산되도록 설계됨(3가지 **계층 구조** 유형의 DNS 서버)

- **<span style="background-color:MistyRose">루트 DNS 서버</span>**
    - ICANN이 직접 관리하는 절대 존엄 서버, TLD DNS 서버 IP 주소 제공
- **<span style="background-color:MistyRose">최상위 레벨 도메인 네임(TLD) DNS 서버</span>**
    - 도메인 등록 기관(Registry)이 관리하는 서버
    - com, org, net, edu, gov 같은 상위 레벨 도메인
    - kr, uk, fr, ca, jp 같은 국가코드 상위 레벨 도메인
    - 책임 DNS 서버에 대한 IP 주소 제공
- **<span style="background-color:MistyRose">책임 DNS 서버</span>**
    - 실제 개인 도메인 IP 주소의 관계가 저장되어 있는 서버
- **<span style="background-color:AntiqueWhite">로컬 DNS 서버</span>**
    - 인터넷 사용자가 실제 쓰는 DNS 서버

EX)

![Untitled](https://user-images.githubusercontent.com/85394884/206618622-8c443282-99c0-4fe2-97e5-982ee324aa7e.png)

1. 웹 브라우저에 www.naver.com을 입력하면, 로컬 DNS 서버에게 “www.naver.com”이라는 호스트 이름에 대한 IP 주소를 요청
2. 로컬 DNS 서버에 해당  IP 주소가 없다면 다른 DNS 서버들과 통신을 시작한다.
    1. 단, 이전에 www.naver.com에 접속한 기록이 있다면, 로컬 DNS 서버에 접속 정보가 캐싱되어 있어 그 즉시 브라우저에 IP 주소를 응답해주고 끝날 수도 있다.
3. 루트 DNS 서버에 “www.naver.com”의 IP 주소 요청
4. 루트 DNS 서버는 “www.naver.com”의 IP 주소를 찾을 수 없어 대신  .com의 주소를 알고 있는 TLD DNS 서버의 주소를 알려줌
5. TLD DNS 서버에 마찬가지로 요청하면 naver.com을 관리하는 책임 DNS 서버의 주소를 알려줌
6. 책임 DNS 서버에 마찬가지로 요청하면 이제 [naver.co](http://naver.com)m 책임 DNS 서버엔 www.naver.com의 IP 주소가 있어 해당 IP 주소를 로컬 DNS 서버에 응답한다.

- 호스트에서 로컬 DNS 서버로 보내는 질의 요청은 호스트 자신을 대신하여 필요한 것들을 얻는 것이므로 ‘**재귀적 질의**’ 라고 한다.
- 로컬 DNS 서버가 루트 DNS 서버, TLD DNS 서버, 책임 DNS 서버에 요청하는 질의는 ‘**반복적 질의**’ 라고 한다.

![Untitled 1](https://user-images.githubusercontent.com/85394884/206618617-5efd27d3-edfd-4738-b22c-0dcc37306c71.png)

<br>

# 메일 서버의 구조 (SMTP와 POP3)

- 인터넷 전자메일 : 비동기적인 통신 매체(상대방 스케줄과 상관없이 자신이 편할 때 메시지를 보내거나 읽는다)

- 인터넷 메일 시스템의 상위 레벨 개념의 주요 요소
    - **사용자 에이전트** : 사용자가 메시지를 읽고, 응답하고, 전달하고, 저장하고, 구성하게 해준다.
        - 대표적으로 MS 아웃룩, 애플 메일, 웹 기반 Gmail
    - **메일 서버** : 메시지를 유지하고 관리하는 메일박스를 가지고 있다.
    - **<span style="background-color:AntiqueWhite">SMTP</span>** : 인터넷 전자메일을 위한 애플케이션 계층 프로토콜 중 하나로 메시지 전송 에이전트라고도 함, 사용 TCP 포트번호는 25번

✅ 송신자 앨리스가 수신자 밥에게 전자메일을 보내는 상황

![Untitled 2](https://user-images.githubusercontent.com/85394884/206618619-bb3fbc28-0bbb-44d5-a83d-13f05768c8d5.png)

1. 앨리스가 메시지를 작성하고 사용자 에이전트에게 메시지를 보내라고 명령한다.
2. 앨리스의 사용자 에이전트는 메시지를 그녀의 메일 서버에게 보내고 그곳에서 메시지는 메시지 큐에 놓인다.
3. 앨리스의 메일 서버에서 동작하는 SMTP의 클라이언트 측은 메시지 큐에 있는 메시지를 보고 밥의 메일 서버에서 수행되고 있는 SMTP 서버에 TCP 연결을 설정한다.
4. SMTP 핸드셰이킹 이후에 SMTP 클라이언트 측은 앨리스의 메시지를 TCP 연결로 보낸다.
5. 밥의 메일 서버에서 동작하는 SMTP 서버 측은 메시지를 수신한다. 
6. 이 메시지를 밥의 메일 박스에 놓는다.
---
7. 밥이 편한 시간에 그 메시지를 읽기 위해 사용자 에이전트를 시동한다.

- SMTP는 메일을 보낼 때 두 메일 서버 사이의 거리가 멀어도 중간 메일 서버를 사용하지 않는다. 수신 측 메일 서버가 죽어 있다면 메시지는 어느 중간 메일 서버에 저장되는 것이 아니라 송신 측 메일 서버에 남아 새로운 시도를 위해 기다린다.

- **<span style="background-color:AntiqueWhite">POP3</span>** : 메일 서버에서 사용자 에이전트로 가져오기 위한 프로토콜, 즉 메시지 엑세스 에이전트, 사용 TCP 포트번호는 110번

- SMTP vs POP3
    - 메시지 전송 에이전트 vs 메시지 엑세스 에이전트
    - 송신 측 컴퓨터에서 수신측 메일 서버로 메일을 보내는 데 사용 vs 수신 측 메일 서버에 있는 메일 박스에서 메일을 가져오는 데 사용