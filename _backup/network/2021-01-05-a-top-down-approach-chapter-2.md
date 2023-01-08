---
title: "[A Top-Down Approach] CH 2. Application Layer"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 2장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-05T16:55:00-05:00
---

## 2.1. Principles of Network Applications

### 2.1.1. Network Application Architectures

#### Client-Server Architecture

Server Host는 항상 작동 중이어야 하며, 영구적으로 할당된 IP를 가지고 있다.

Client Host는 Server Host와 통신할 뿐, Client들 끼리 직접적으로 통신하지 않는다. Server와 간헐적으로 연결이 되며, 동적 IP를 할당받는다.

Server Host가 Client Host의 요청을 처리해주는데, 요청 트래픽이 많은 경우 하나의 Server Host로 감당하기 어려워진다. 거대 컨텐츠 제공자들은 대규모 서비스 제공을 위해 여러 개의 데이터 센터를 가지고 있다.

#### P2P(Peer-To-Peer) Architecture

별도의 Server에 크게 의존하지 않고 End System들끼리 직접 통신하면서 서로의 요청을 주고 받거나 서비스를 제공한다. 이 때, End System을 Peer라고 칭한다.

Peer는 서비스를 요청(다운로드)하면서도 동시에 다른 Peer에게 서비스를 제공(업로드)하기 때문에 자가 확장성(Self-Scalability)이 있다. 그러나 별도의 Server에 크게 의존하지 않고 인터넷을 통해 End System들끼리 통신하는 만큼 관리가 복잡하다.

### 2.1.2. Process Communicating

Process란 한 Host 내부에서 실행 중인 프로그램이다. 하나의 Host 내부에서 동작하는 두 Process가 통신할 때는 주로 OS의 규칙을 따른다. 하지만 Network 측면에서는 서로 다른 두 Host의 Process들끼리 메시지를 교환하며 통신하는 것을 의미한다.

#### Client and Server Processes

Client Process란 통신을 시작하는 프로세스이다. 다시 말해, 먼저 접촉을 시도하는 쪽이다.

Server Process는 접촉을 기다리고 요청을 응대하는 프로세스이다. P2P 구조에서는 하나의 Peer가 Client 및 Server 등 두 가지 역할을 모두 할 수 있다.

#### The Interface Between the Process and the Computer Network

Application Layer는 개발자에 의해 제어되는 프로그램 영역이다. 메시지를 생성하고 Transport Layer에게 해당 메시지를 다른 End System까지의 전달을 요청한다.

반면 Transport Layer 이하 하위 계층들은 OS에 제어되는 영역이다. Socket은 Application Layer와 Transport Layer 사이의 API로서 문과 같은 역할을 한다. Application Layer의 Process는 Socket을 통해 메시지를 보내거나 전달 받는다.

#### Addressing Processes

네트워크에서의 커뮤니케이션이란 엄밀하게 두 Host가 아닌 두 Process끼리 이루어진다.

Application Layer에서 생성한 메시지가 패킷화되서 전송될 때, 해당 패킷이 도착할 목적지 Host의 위치는 32비트 IP 주소로 식별할 수 있다. 이 때 Port 번호를 통해 목적지 End System 내부에 존재하는 특정 Process를 식별할 수 있다.

HTTP 프로토콜은 80 포트 번호를 사용한다.

### 2.1.3. Transport Services Available to Applications

어플리케이션의 목적에 따라 어떤 종류의 트랜스포트 서비스를 원하는지 상이해진다.

은행 거래 등 데이터 무결성(Data Integrity)가 중요한 어플리케이션의 경우, 신뢰성있는 데이터 전송이 필요하다. 반면 오디오 등의 어플리케이션은 약간의 데이터 손실이 용인된다.

인터넷 전화, 실시간 상호작용 게임은 데이터 송수신에 걸리는 지연시간인 Ping이 낮아야 한다.

멀티미디어는 단위 시간당 일정량의 프레임(예를 들어 60 프레임)을 보장해야 사용자가 불편함을 느끼지 않기 때문에, 단위 시간당 전달받아 처리할 데이터 양의 최소 기준을 효과적으로 충족하는데 주안을 둔다.

어떤 어플리케이션은 보안이 중요할 것이고, 어떤 어플리케이션은 위 사항들이 딱히 중요하지 않을 수 있다.

### 2.1.4. Transport Services Provided by the Internet

인터넷은 크게 TCP와 UDP라는 두 가지 Transport Protocol을 제공하고 있다.

#### TCP(Transmission Control Protocol)

* Connection-Oriented Service
  * Application의 메시지가 전달되기 전에 Handshaking을 통해 Client와 Server의 Process간의 연결을 먼저 셋업한다.
* Flow Control
  * Sending쪽의 TCP와 Destination 쪽의 TCP는 모두 Buffer를 가지고 있다.
  * Sending TCP는 Application이 보내는 메시지의 패킷들을 Buffer에 담아 관리하다가 보내고, Destination TCP는 전달받은 메시지의 패킷들을 Buuffer에 담아 관리하다가 Application으로 올려 보낸다.
    * Sending TCP의 데이터 송신 속도가 Destination TCP의 데이터 수신 처리 속도보다 빠른 경우, Buffer가 꽉 차서 패킷 손실이 발생할 수 있다. (Router의 Queueing Dleay와 유사하다.)
    * TCP 프로토콜은 이러한 위험을 줄이고자 송신 측의 데이터 전송량을 수신 측의 상태에 따라 조절해준다.
* Congestion Control
  * Network Layer에서 하나의 노드가 네트워크를 과부화하는 것을 방지한다.
  * 예를 들어 두 노드 라우터 사이의 Link에 많은 Trrafic이 몰려 Queueing Delay가 생기는 경우 혼잡을 제어한다.
  * Flow Control와 유사하지만, Flow Control은 End Node와 관련된 내용이다.
* Reliable Transport
  * Process는 TCP를 통해 모든 데이터가 에러 없이 순서대로 전달됬음을 신뢰할 수 있다.
  * 한 쪽의 Application이 소켓으로 바이트 스트림을 보내면, TCP는 중복이나 누락없이 해당 바이트 스트림을 받는 쪽 Application의 Socket으로 전달하는 것이 보장된다.
  * 데이터 무결성(Data Integrity).

커넥션 관리에 오버헤드가 발생할 수 있다. 특히, 메시지를 한 번 받으면 끝나는 커넥션이라면 배보다 배꼽이 커지는 상황이 발생한다. 실시간 Timing, Security, 최소 작업 처리량(Throughput) 보장이 중요한 어플리케이션에서는 효율을 낼 수 없을 수도 있다.

또한 데이터 무결성이 중요한 Application이라면 TCP의 Reliable Transport 이후 Application Layer에서도 데이터에 대한 유효성을 검사할 것이다. Protocol을 캡슐화하면서 두 계층의 데이터 무결성에 대한 기능들이 중복되기 때문에, UDP를 사용할 수도 있다.

미디어 어플리케이션은 단위 시간당 전달받아 처리할 데이터 양이 최소 기준을 충족해야 하는 경우가 많다. (예 : 동영상 어플리케이션은 60 프레임.) 이 때 Flow나 Congestion Control을 하는 TCP는 데이터를 중간에 전달하지 않고 홀드할 수 있기 때문에 원하지 않는 결과가 발생할 수 있다.

#### UDP(User Datagram Protocol)

UDP는 가벼운 프로토콜로서 최소한의 서비스만을 제공한다. 모든 데이터가 순서대로, 정상적으로 도착했는지 보장할 수 없는 Unreliable Data Transfer이다.

### 2.1.5. Application-Layer Protocols

Application이 만들어내는 메시지 또한 특정 프로토콜을 준수한다. Application 레이어의 프로토콜은 다음 내용 등을 정의한다.

* 교환되는 메시지 유형 : 요청 / 응답
* 메시지 신택스 : 어떤 필드가 있고, 어떻게 구분되어져야 하는지.
* 메시지 해석은 어떻게 해야하는지.
* 어떤 식으로 응답하거나 처리하는지 등과 관련된 규칙.

공개되어 있는 프로토콜은 RFCs, HTTP, SMTP 등이 있으며, Skype 등은 비공개 프로토콜을 사용한다.

<br>

## 2.2. The Web and HTTP

웹 페이지는 여러 HTML, JPEG, PDF 등 여러 레퍼런스 Object들을 포함하고 있는 Base HTML File로 구성되어 있다. 해당 오브젝트들은 {hostname/path} 패턴의 URL을 통해 접근한다.

### 2.2.1. Overview of HTTP

HTTP(Hypertext Transfer Protocol)는 웹 어플리케이션 레이어의 프로콜이다. Client-Server Model을 따르며, Client는 요청 메시지를 보내고 응답을 받으면 웹 자원 오브젝트들을 보여주는 Brower이다. Server는 Client의 요청에 맞는 오브젝트들을 응답으로 보내주는 웹 서버이다.

HTTP는 Data Integrity가 중요하기 때문에 TCP를 사용한다. Client가 Server와 소켓을 만들어 연결을 맺으며, 포트 번호는 80번이다. 이후 Server가 승인하게 되고, HTTP 메시지를 통해 브라우저와 웹 서버간 메시지 교환이 발생한다. 작업이 마무리되면 TCP 커넥션이 닫히게 된다.

HTTP는 기본적으로 무상태(Stateless) 프로토콜이다. Server는 Client가 이전에 보낸 Request 히스토리를 기억할 수 없다. 만약 저장하는 경우, 크래시가 발생했을 때 Server와 Client의 View를 일치시키는 것이 매우 복잡해지기 때문이다.

### 2.2.2. Non-Persistent and Persistent Connections

#### Non-Persistent HTTP

HTTP 1.0의 경우에 해당한다. 1개의 TCP 연결은 1개의 Object만을 전송할 수 있는 프로토콜이다.

작동 중인 서버는 대기 상태에서 클라이언트의 연결 요청이 들어오면 이를 승인하다. 클라이언트가 HTTP 요청 메시지를 작성해 TCP 소켓을 통해 보낸다. 서버는 요청된 Object를 포함하는 응답 메시지를 작성하는데, 처음에는 Base HTML 파일만 포함한다. 이후 TCP 소켓을 통해 응답 메시지를 클라이언트에게 보내고 연결이 자동으로 끊긴다.

클라이언트는 HTML 파일을 포함하는 메시지 응답을 받았지만, HTML에 포함된 N개의 오브젝트 레퍼런스를 발견하게 된다. Non-Persistent HTTP의 경우, 해당 레퍼런스 오브젝트들에 대해 커넥션을 열고 Object를 받고 커넥션을 닫는 행위가 N번 반복하게 된다.

RTT(Round Trip Time)이란 작은 패킷이 클라이언트에서 서버로 갔다가 다시 클라이언트에게 돌아오는 시간을 의미한다. Non-Persistent HTTP에서 클라이언트가 서버와 연결되고 최초의 Base HTML 파일을 응답 받기 까지 다음과 같은 시간이 소요된다.

* 1 RTT : 클라이언트가 TCP 커넥션 생성 요청을 보내고 다시 응답받는 시간.
* 1 RTT : HTTP 요청 메시지를 보내고 HTTP 응답 메시지를 받는 시간.
* File Transmission Time : 요청에 대한 여러 File 들을 밀어 넣는 Transmission 시간.
* 즉 2RTT + File Transmission Time이 소요된다.

만약 Base HTML 파일에 추가로 로드해야 할 레퍼런스 Object가 N개가 있다면, 이후 클라이언트가 서버와 연결을 맺고 레퍼런스 Object 자원을 요청 및 응답받는 행위가 N번 반복되게 된다.

따라서 Non-Persistent HTTP에서는 웹 페이지를 띄우기 위해 (N+1)(2RTT + File Transmission Time)만큼의 시간이 소요된다.

다만 이를 해결하기 위해 Parallel TCP 연결을 맺는 경우가 있다. 레퍼런스 Object가 N개라면 브라우저는 N개의 TCP 연결을 동시에 맺는다. 거의 동시에 비동기적으로 TCP 연결 + 클라이언트의 Object 요청 + 서버의 Object 응답 작업이 이루어지기 때문에, 순차적인 TCP 연결보다 RTT가 줄어들게 된다.

다만 Transport Layer에서 Socket을 할당하는 작업(TCP 연결)은 OS의 책임이기 때문에, 연결을 여러 개 만드는 경우 OS 오버헤드가 발생한다. 각각의 TCP 커넥션마다 소켓과 버퍼를 할당해야 하기 때문이다.

#### Persistent HTTP

1개의 TCP 연결이 N개의 Object를 전송할 수 있는 프로토콜이다. 처음 TCP 연결 이후, 서버가 클라이언트에게 Object를 담은 응답을 보내도 TCP 연결이 계속 유지된다.

클라이언트 브라우저가 처음 Base HTML을 받고 이후 여러 레퍼런스 Object들에 대해 Request 메시지를 계속 보내는데, 이 차이가 무시할 수준이라면 RTT는 1에 수렴하게 된다.

### 2.2.3. HTTP Message Format

HTTP 메시지는 Request와 Response 두 가지로 구분된다.

#### HTTP Request Message

* Request Line : GET, POST 등의 메서드 타입과 요청 URL.
* Header Lines : Host, User-Agent, 인코딩, 커넥션, 브라우저 정보, 언어, 컨텐츠 타입 등.
* 마지막은 carraiage return(\r\n)이 오는데, 필요하다면 밑에 Entity Body가 추가될 수 있다.

* Metho Types
  * POST: Form Input 등 입력값을 Entity Body에 담아 요청한다.
  * GET : URL 필드에 입력값을 붙여서 요청한다.

HTTP/1.0에서는 GET과 POST 및 HEAD 메서드만이 존재했다. HEAD는 Response 메시지는 보내되, 요청한 Object는 보내지 않는 메서드이다. 이후 HTTP/1.1에서는 PUT과 DELETE 메서드가 추가되었다.

#### HTTP Response Message

* Status : HTTP 버전과 Status 코드 및 메시지.
* Header Lines : 날짜, 시간, 수정 시간, 커넥션(TCP Close or Keep-Alive 등), 컨텐츠 길이, 타입 등.
* Body : 요청에 대한 Data와 HTML FILE 등.

### 2.2.4. User-Server Interaction: Cookies

HTTP는 무상태 프로토콜이기 때문에 간단해지고 오버헤드가 줄어들지만, 유저 히스토리가 없기 때문에 유연한 서비스 제공이 어려워진다.

이를 극복하고자, 서버 사이드는 Cookie를 통해 유저에 대한 히스토리를 저장해놓는다.

* 동작 예시
  * 클라이언트에서 최초 HTTP 요청이 들어오면, 서버는 요청을 처리할 때 특정 ID를 만들고 DB에 해당 ID에 해당하는 엔트리를 생성한다.
  * HTTP 응답 메시지의 헤더 라인에 set-cookie 정보를 추가한다.
  * 쿠키 파일은 클라이언트의 브라우저에 저장 및 관리되어진다.
  * 이후, 클라이언트는 서버에 HTTP 요청을 보낼 때 마다 메시지에 쿠키 헤더라인을 추가한다.
  * 서버는 해당 쿠키 아이디를 통해 DB에 접근하여 요청을 수행하고 응답을 한다.
    * 쿠키 만료 기간 전에 클라이언트가 서버에 다시 요청을 보낼 때, 그 때에도 쿠키 아이디가 있으면 DB에 접근하고 요청을 수행한다.

Cookie를 통해 User의 특정 세션(로그인) 상태 및 Authorization을 유지할 수 있다. 웹 사이트에서 여러 페이지에 걸쳐 유저의 과거 요청 정보를 보여주는 경우(쇼핑 카트, 로그인 상태, 아이디 및 비밀번호 저장 등) 유용하게 사용한다.

**주의** : 여기에서는 엄밀하게 Cookie와 Session을 구분하지 않는다.

### 2.2.5. Web Caching

Web Cache는 Proxy Server라고도 부르며, 오리지널 서버를 수반하지 않고도 클라이언트의 요청을 충족시키는 역할을 한다.

Client와 오리지널 Server 사이에 Proxy Server가 위치해있다. 유저는 캐시를 통해 웹에 접근하도록 브라우저를 설정한다.

A 클라이언트가 특정 오브젝트에 대한 최초의 HTTP 요청을 보냈을 때, 웹 캐시 서버는 오리지널 서버로부터 해당 오브젝트를 받아 A 클라이언트에게 돌려주면서 이를 캐싱(저장)해놓는다.

이후 B, C 등 다른 클라이언트가 해당 오브젝트를 요청했을때 요청들은 Original Server까지 가지 않고 중간의 웹 캐시 서버가 요청에 대한 응답을 처리해준다.

웹 캐시 서버는 오리지널 서버와의 관계에서는 클라이언트지만, 클라이언트와의 관계에서는 서버의 역할을 한다. 주로 대학이나 회사 등의 ISP가 캐시를 설치한다. 캐시를 통해 기관의 Router와 ISP 사이의 Access Link(POP)에 대한 트래픽이 감소하며 비용이 절감되고, 클라이언트 요청에 대한 응답 시간 또한 감소한다.

또한 자금이 부족한 컨텐츠 제공자들은 많은 ISP를 사용하는데 어려움이 많기 때문에, 많은 네트워크 인프라 없이 쉽게 컨텐츠를 제공하고자 사용하게 된다.

### 2.2.6. Conditional Get

이러한 웹 캐시 서버는 자신이 가지고 있는 정보가 정말 최신(up-to-date) 정보인지 확인하기 위해 Conditional Get을 한다.
* 캐시 서버는 클라이언트가 보낸 요청의 응답에 해당하는 캐시 카피본이 있으면, 특정 시간 이후 수정이 되었는지 등 날짜와 시간을 캐시 카피본에 붙여 오리지널 서버에 요청한다.
* 요청받은 시간 이후 업데이트 된 적이 없다면, 오리지널 서버는 304 Not Modified 응답 코드를 반환한다.
  * 이 때, 오리지널 서버는 캐시 서버에게 해당 Object 보내지 않으며 캐시 서버가 자신이 가진 Object를 클라이언트 브라우저에게 보낸다.
* 만약 업데이트 이력이 있다면 오리지널 서버는 200 OK 응답 코드와 더불어서 업데이트된 최신 Object를 반환한다.

캐시가 오리지널 서버에 최신성을 체크하는 간격에 따라서, 데이터의 최신성과 오버헤드(기관의 Router와 ISP 사이의 Link POP)의 Trade-Off가 존재한다.

<br>

## 2.3. Electronic Mail in the Internet

User Agent는 메일을 읽고 쓰는 Reader이며, Mail Server는 보내거나 받는 메시지를 전부 저장한다. 이 때, Mailbox는 유저가 받는 메시지를 저장하며, Message Queue는 유저가 현재 보낸 메시지를 저장하는 공간이다. 메일 박스는 Queue가 아닌 사용자 별로 존재한다.

E메일 프로토콜은 SMTP(Simple Mail Transfer Protocol), POP3, IMAP, HTTP 등 다양하다. 클라이언트가 보내는 쪽 서버에 SMTP 프로토콜로 메일 보내고, 보내는쪽 서버는 받는쪽 서버에 SMTP로 메시지를 전송한다. 받는 쪽 서버는 POP3, IMAP, HTTP 등을 통해 클라이언트에게 받는 메일을 최종적으로 전달한다.

### 2.3.1. SMTP(Simple Mail Transfer Protocol)

보내는 쪽 메일 서버에서 받는 쪽 메일 서버로 메시지를 전달하는 프로토콜이다.

여러 클라이언트와 메일 서버들이 엉켜있기 때문에, 클라이언트-서버 관계는 접촉 시도-응대 관계에 따라 달라진다. TCP 프로토콜을 사용하며 포트 번호는 25번이다.

클라이언트는 아스키 텍스트 등으로 커맨드를 보내고, 서버는 상태 코드 및 메시지로 이루어진 응답을 보낸다. 현재는 텍스트 뿐만 아니라 다양한 멀티 미디어를 보낼 수 있다.

Handshaking(Greeting) , Transfer, Closure 등 세 단계로 구분된다. 또한 Persistent Connection을 사용하며, 메시지의 끝을 확인하기 위해 CRLF를 사용한다.

HTTP는 클라이언트가 서버에 접촉하여 자료를 가져오는 Pull Protocol이라면, SMTP는 Push Protocol이다. Multipart 메시지에 여러 오브젝트들이 저장된다. 두 프로토콜 모두 Ascii 커맨드 및 응답 상호작용과 상태코드 등의 공통점이 있다.

### 2.3.2. Mail Access Protocols

받는 쪽 서버가 최종 유저에게 메시지를 전달할 때 사용하는 프로토콜이다. G-Mail 등은 HTTP를 사용한다. 과거에는 POP 혹은 POP를 개선한 IMAP 등을 사용하기도 했다.

#### POP 3 Protocol

* Authorization Phase : 클라이언트가 유저 및 패스워드 커맨드를 보내 인증하고, 서버가 응답하는 단계이다.
* Transaction Phase : Client가 메일 리스트를 조회하고, 번호를 통해 개별 메시지를 상세 조회하고, 메시지를 삭제하거나 프로토콜을 종료한다.

수신된 메일이 있으면 클라이언트 쪽으로 다운로드 하게 되는데, 과거에는 Download-and-Delete여서 한 번 읽으면 다른 클라이언트 Local PC에서는 메일을 확인하지 못했다. Download-and-Keep 등의 개선 방안이 나왔다.

#### IMAP

모든 메시지를 하나의 장소(서버)에 저장한다. 유저가 메일 박스 속의 메시지들을 관리할 수 있다. 또한 한 세션에서 메일과 관련된 작업을 해도, 다음 세션에도 그 결과물들이 유지된다.

<br>

## 2.4. DNS—The Internet’s Directory Service

인터넷 호스트, 라우터는 32비트의 IP 주소를 가지고 있다. 그러나 숫자 주소보다 이름이 인간에게 직관적이기 때문에, IP 주소에 Host Name을 붙이기 시작했다.

DNS(Domain Name Service)는 분산 Database로 IP와 Host Name간의 매핑 정보를 저장한다. DNS 자체는 Network Layer가 적합해 보이지만, Network Layer는 패킷을 전달해주는 핵심 레이어다. 모든 복잡성은 Edge한 부분으로 몰아내자는 철학때문에 DNS는 Application Layer에 속하게 되었다.

### 2.4.1. Services Provided by DNS

DNS는 다음과 같은 기능을 수행할 수 있다.
* Host Name을 IP 주소로 번역한다.
* Host Aliasing을 제공한다.
  * 하나의 아이피는 여러 Host Name을 가질 수 있다.
  * Alias 이름을 실제 외부에 알려진 이름으로 치환해준다.
  * Mail Server로서 메일 주소 도메인을 알려준다.
  * Load Distribution.
    * 모든 클라이언트 요청을 1개 서버가 담당하게 되면 트래픽 부하가 심해진다.
    * 하나의 어플리케이션이 여러 웹 서버(즉, 여러 IP)를 가지게 하고, 클라이언트들이 하나의 호스트 네임으로 접속 및 요청을 해도 DNS를 통해 요청을 여러 웹 서버 IP로 분산시킨다.

### 2.4.2. Overview of How DNS Works

DNS를 중앙화하지 않고 분산하는 이유는 다음과 같다.
* 한 부분이 오작동을 일으키면, 전체 시스템이 먹통이 될 수 있다.
* Traffic Volume이 한 곳으로 너무 집중되어 커진다.
* DB를 중앙화해버리면, 거리가 먼 지역들은 응답 속도가 느리며 리소스 낭비가 심해진다.
* 유지 보수.

DNS는 계층적이고 분산된 Database이며, 이런 DNS를 관리하는 서버가 있다.

* Root DNS Servers.
* TLD(Top-Level Domain) Servers.
  * com
  * org
  * edu
  * jp, kr 등 국가 레벨 도메인.
* Authoritative DNS Servers.
  * 각 회사 혹은 기관의 DNS Servers.

또한 세 계층에 속하지 않는 Local DNS Server가 존재한다. Default DNS Server 혹은 Proxy DNS Server라고 불리는데 주로 회사와 대학 등 Residental ISP가 만든다.

* 호스트가 DNS Query를 만들면 Local DNS Server에 요청이 간다.
* Proxy Server처럼 최근 Name-to-Address 번역 맵핑 결과를 보유하고 있으며, 결과가 없는 경우 최상위 Root DNS Server에 쿼리를 던진다.
  * 해당 맵핑 결과가 최신인지 Conditional Get한다.

#### Iterated Query

처음 Local DNS Server에 특정 호스트 이름에 대한 요청을 보내고, 정보가 없으면 Root DNS Server에 묻는다.

Root DNS 서버는 ".edu TLD 서버에게 물어봐" 응답을 주고, Local DNS Server는 해당 TLD 서버에 요청을 보낸다. TLD 서버는 "xxx Authoritative DNS 서버에게 물어봐"라는 응답을 준다.

마침내 Local DNS Server가 해당 기관 DNS 서버에 질의하면, 해당 기관의 DNS 서버는 관리하고 있는 Host Name들 중 일치하는 IP 맵핑 결과를 응답해준다.

#### Recursive Query

Local DNS Server가 Root DNS Server에 호스트 이름을 물어보면, Root DNS Server가 TLD 서버에게 직접 해당 호스트 이름을 요청하여 맵핑 아이피 정보를 받아오고, 최종적으로 이를 Local DNS Server에게 제공하는 방식이다.

Root DNS Server와 TLD Server에 부하가 커진다.

#### DNS Caching

Local DNS Server가 호스트 이름-IP 주소 맵핑 정보를 캐싱하고 있으면, 요청이 Root DNS Server까지 안 가고도 응답이 가능하다. TLD Server를 캐싱하는데, 캐싱 유지 시간(TTL) 안에 DNS가 변경되면 최신 호스트 이름-IP 맵핑 결과가 아닐 수 있다.

### 2.4.3. DNS Records and Messages

DNS 프로토콜에서 Query와 Reply 모두 동일한 형식의 메시지다.
* Flag, Query 방식, Identification 등.
* 밑의 가변 길이의 필드 (Name, Type, RR, Records 등).

DNS란 Resource Records를 저장하고 있는 분산 DB인 것이다. Resource Record는 (name, value, type, TTL) 등의 맵핑 정보를 보유하고 있다. 다음 타입들은 RR에서의 name-value 맵핑 관계에 대한 타입 유형이다.

* Type A : HostName - IP 주소 맵핑 관계.
* Type NS : Domain - Domain의 Hostname 맵핑 관계.
* Type MX : Name - Nmae 관련 메일 서버 맵핑 관계.
* Type CNAME : Alias Name - Canonical Name 맵핑 관계.

<br>

## 2.5. Peer-to-Peer Applications

BitTorrent 등이 대표적인 P2P 구조의 어플리케이션이다. 해당 구조에서의 End System들을 Peer라고 칭한다. 자가확장성이 좋다.

### 2.5.1. P2P File Distribution

일반적인 Client-Server 모델에서 파일의 크기가 F고 클라이언트가 N개 일 때, 서버가 파일을 배분하는데 걸리는 시간은 다음과 같다.

Distribution Time >= max{N*F/(서버의 파일 업로드 속도), F/(가장 낮은 클라이언트 다운로드 속도)}

사용자의 수가 많아질 수록 Distribution 시간이 선형적으로 증가하는 구조이다.

P2P 환경에서의 파일 배분에 걸리는 시간은 다음과 같다.

Distribution Time >= max{F/(서버의 파일 업로드 속도), F/(가장 낮은 클라이언트 다운로드 속도), N*F/(서버의 파일 업로드 속도 + 모든 피어들의 업로딩 속도의 합)}

분자가 늘어날 수록 분모도 늘어나기 때문에, 사람이 많아져도 Client-Server 구조와 다르게 파일 배분에 걸리는 시간의 증가폭이 낮다.

#### BitTorrent

파일을 256kb 청크로 나누고, 피어들이 청크를 업로드 및 다운로드하게 한다. 토렌트란 이 피어 그룹들이 서로 필요한 청크들을 교환하는 것이며 피어란 클라이언트와 서버의 역할을 모두 한다.

Tracker 서버는 토렌트에 참여하고 있는 피어들 중 내가 필요한 청크를 가진 피어를 알려준다. 이를 제외하면 Client-Server와 다르게 Server에 크게 의존하지 않는다.

#### Peer Churn

피어는 예고없이 접속했다가 끊겼다가 하는 특징을 가지고 있다. 이런 특성 때문에, 불성실한 피어들이 다수 존재할 수 있다.

Request Chunks란 이웃하고 있는 Peer들 중 가장 희소한 청크를 가진 Peer를 높은 교환 우선순위로 선정하는 것이다. 이를 통해 해당 희소한 청크를 이웃 피어들과 가장 먼저 교환하게 한다. Peer는 지속적으로 이웃 피어들이 가진 청크 목록에 대해 요청하며 확인한다.

또한 피어는 특정 시간마다 나에게 파일을 가장 잘 공급해주는 성실한 피어들을 선정하고, 그들에게 청크를 우선 공급해준다. 청크 공급에 성실하게 참여하는 피어일 수록, 더 잘 공급받을 수 있도록 하는 원리이다. 부작용을 방지하기 위해, 특정 시간마다 랜덤으로 피어를 찾아 교환하기도 한다. (tit-for-tat).

<br>

## 2.6. Socket Programming

소켓이란 Application과 Transport Layer 사이의 인터페이스이다.

### 2.6.1. UDP

서버가 먼저 작동하고 있으며, 클라이언트와 서버 사이의 핸드쉐이킹을 통한 선제 연결은 없다.

Sender는 IP 도착 주소와 포트 정보를 모든 패킷에 부착한다. Receiver는 패킷에서 Sender의 주소와 포트 번호를 추출해낸다.

* 서버는 먼저 소킷을 만든다.
* 클라이언트도 소킷을 만들고, Datagram에 서버의 IP와 포트 번호를 붙인다.
* 클라이언트 소킷을 통해 Datagram을 전송한다.
* 서버는 서버 소킷으로부터 Datagram을 읽는다.
* 서버는 서버 소킷을 통해 응답 Datagram을 작성해 보낸다.
  * 이 때, 클라이언트 주소와 포트 번호를 명시해야 한다.
* 클라이언트는 소킷을 통해 응답 Datagram을 읽는다.

### 2.6.2. TCP

클라이언트가 서버에게 먼저 접촉한다.

* 서버는 Door 소켓이 생성된 상태여야 한다.
* 클라이언트는 서버에 접촉하기 위해 TCP 소켓을 만든다.
  * 또한 컨택할 서버 프로세스에 대한 포트 정보, IP 등을 명시한다.
* 클라이언트 TCP 소켓이 서버 TCP와 연결을 맺는다.
* 클라이언트가 접촉하면 서버는 해당 클라이언트를 위한 새로운 소켓을 만든다.
  * A라는 클라이언트와 소켓 통신 중에, B와 C 등 다른 클라이언트에 대해서도 소켓을 열어 다중 통신할 수 있다.
  * 1:1 파이프 통신이기 때문에 서버는 클라이언트 도착지를 명시하지 않아도 된다.
    * 반면 UDP는 각 메시지가 독립적이고 답장만 해주면 끝난다.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
