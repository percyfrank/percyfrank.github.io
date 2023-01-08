---
title: "[A Top-Down Approach] CH 1. Computer Networks and the Internet"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 1장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-03T16:55:00-05:00
---

## 1.1. What Is the Internet?

인터넷은 디바이스들을 연결하는 컴퓨터 네트워크이다. 전통적인 의미의 디바이스는 컴퓨터에 국한되었으나, 오늘날에는 스마트폰과 TV 등 다양한 기기들도 포함된다.

이러한 디바이스들을 Host(네트워크 어플리케이션을 실행하는 주체) 혹은 End System(인터넷의 가장 자리에 위치한 존재)이라 칭한다.

Host들은 물리적 매체(동축 케이블, 전선, 라디오 등)들로 이루어진 Communication Link들에 의해 네트워크로 연결된다. 이러한 통신 링크들은 대역폭(전송 속도)가 상이하다.

Host들이 통신할 때, 데이터는 Packet이라는 정보의 묶음으로 전달된다. 이러한 Packet이 통신 링크를 타고 이동하다가 만나는 Router는 길을 찾아주는 역할을 한다. Router는 Packet의 목적지까지의 경로를 분석한 뒤, 패킷 교환(Switch)을 통해 다음 Router로 패킷을 Forward해준다.

End System은 ISP(Internet Service Provider)를 통해 인터넷에 접근할 수 있다. 케이블 모뎀이나 DSL 등 다양한 형태의 네트워크 접근을 제공하거나, 혹은 콘텐츠 제공자에게 인터넷 접근을 제공한다. ISP 또한 계층적인 구조를 가지며, 계층들간 ISP들끼리 상호 연결되어 있다.

Protocol이란 메세지의 송수신 제어에 대한 통신 규약이다. 즉, 인터넷 상에서 송수신할 메시지의 포맷과 순서 및 취해야 할 행동 등을 정의해두었다. 인터넷 구성 요소들은 Protocol을 준수하며 통신한다. 대표적으로 TCP/IP 등이 있다.

<br>

## 1.2. Network Edge

네트워크는 크게 Network Edge, Access Network, Network Core 등 3가지로 분류할 수 있다. Edge에 위치한 End System은 크게 클라이언트와 서버로 구분되어진다.

### 1.2.1. Access Network

End System이 다른 End System으로 연결되는 경로에는 여러 Router들이 존재한다. Access Network는 하나의 End System과 해당 End System이 경로에서 가장 처음 만나는 Edge Router 사이를 물리적으로 연결해주는 네트워크이다.

#### Home Access

* DSL(Digital Subscriber Line)
  * 전화 회사 등이 고객의 ISP가 되어 Access Network를 제공해준다.
  * 각 가정은 DSL Modem을 가지고 있으며, 전화 및 컴퓨터 데이터가 Splitter를 거쳐 전화 회사의 Central Office로 전송된다.
    * 각 가정마다 서로 다른(Dedicated) 회선을 사용한다.
  * 전화 회사는 DSLAM이라는 멀티 플렉서를 통해 전화는 전화국으로, 컴퓨터 데이터는 ISP로 연결해준다.
  * 가정은 주로 Data의 소비자이기 때문에 업로드하는 Upstream Bandwidth보다 다운로드하는 Downstream Bandwidth이 더 넓다.
* Cable Network
  * 케이블 TV 회사의 회선을 통해 가정에 Access Network를 제공해준다.
  * 각 가정은 Cable Modem을 가지고 있으며, 방송 및 컴퓨터 데이터가 Splitter를 거쳐 Cable Headend로 전송된다.
    * 케이블 TV 회사의 CMTS라는 멀티 플렉서를 통해 ISP로 연결된다.
  * Cable Headend들을 연결할 때 사용되는 통신 링크들의 소재들 때문에 HFC(Hybrid Fiber Coax)라고도 부른다.
  * TV 방송 데이터는 브로드캐스트를 사용하기 때문에 가정들이 하나의 회선을 공유(Shared)한다.
  * DSL처럼 Upstream과 Downstream이 비대칭적이며, 회선을 공유하기 때문에 사용자의 수마다 속도가 다를 수 있다.
* FTTH(Fiber To The Home)
  * 광섬유 회선을 회사에서 가정으로 직접 연결하는 방법이다.
  * 이론적으로 높은 속도를 기대할 수 있으나, 비싼 비용이 단점이다.
* 인공위성

#### Enterprise(And Home) Access

* Ethernet
  * LAN(Local Area Network)을 사용한다.
  * 학교, 회사 등 기관은 End System이 많다.
  * 한 층 혹은 한 건물에 Ethernet Switch가 모든 End System을 연결해주고, Ethernet Switch들은 해당 기관의 Router로 연결된다.
  * 기관은 내부 장비의 규모가 크고 데이터가 많기 때문에, DSL 혹은 Cable을 사용하지 않고 ISP Router로 직접 연결(Dedicated)한다.
* Wireless Access Network
  * WiFi Access-Point를 포함하여 여러 End System들이 가정에서 하나의 Router에 묶이고, DSL이나 Cable과 연결된다.
  * 3G, LTE 등은 통신사의 기지국의 범위 내에서 사용이 가능하다.
  * 공유되는 네트워크들이다.

### 1.2.2. Physical Media

비트 정보는 물리적 매체를 통해 전파된다.

#### Guided Media

Twisted-Pair Copper Wire, Coaxial Cable, Fiber Optics 등이 있다. 각 매체의 물리적 특성에 따라 전송 속도(대역폭)이 다르다. 대역폭이 넓을 수록 한 번에 많은 비트 정보를 전송할 수 있기 때문에, 전송 속도가 빠르다.

#### Unguided Media

* Terrestrial Radio Channel
  * 전자기적 스펙트럼을 통해 신호를 전송한다.
  * 어느 범위까지 전달되는지에 따라 크게 3가지로 구분된다.
  * 전달 거리, 장애물, 다른 전자기적 스펙트럼 등의 환경적인 요소에 따라 간섭을 받는다.
* Satellite Radio Channel

<br>

## 1.3. Network Core

### 1.3.1. Packet Switching

End System들이 통신할 때 데이터(메시지)는 Packet이라는 정보의 묶음으로 쪼개진다. 너무 큰 데이터(메시지)를 그대로 전달하게 된다면 하나의 Host가 Link를 오래 점거할 수 있기 때문이다.

Packet은 통신 Link를 통해 전달된다. Packet이 목적지에 도달하기 위해, 패킷 스위치(Router나 Link-Layer Switch)는 다음 경로의 Router로 Packet을 Forward한다. 다시 말해, 다음 현재 Router는 다음 Router로 향하는 통신 링크에 패킷을 밀어넣는다.

출발 지점의 Host가 L bit의 데이터를 전송할 때, 링크의 전송 속도가 R bits/sec이면 패킷을 전송하는데 걸리는 시간은 L/R 초이다.

#### Store-and-Forward Transmission

* 패킷 스위치는 Queue 구조의 Buffer를 가지고 있으며 패킷 1개를 완전히 전달받은 이후에야, 그 다음 Router로 향하는 링크에 해당 패킷의 첫 비트를 전송하기 시작한다.
* 패킷을 시작 지점에서 목표 지점으로 전송하는데 걸리는 딜레이는 N(링크 개수) * L/R이다.

#### Queuing Delays and Packet Loss

* 패킷 스위치는 여러 링크들과 연결되어 있으며, 다음 링크로 전송할 패킷들을 저장해놓은 Output Buffer가 있다.
* 원활한 Output 작업이 진행되지 않으면, Input으로 들어온 패킷들은 대기하게 되며 Buffer에 패킷들이 쌓이기 시작한다.
  * 현재 Router에서 다음 Router로 향하는 통신 Link의 대역폭(처리 속도)이 작은 경우.
    * Buffer에서 패킷이 빠져나가는 속도보다 유입되는 패킷의 트래픽이 많은 경우 발생한다.
* Buffer 사이즈는 유한대이기 때문에, Buffer가 꽉 찬 경우 새로 전송 받은 패킷 혹은 Buffer에 저장된 패킷이 손실될 수 있다.

#### Forwarding Tables and Routing Protocols

데이터 송수신이 이루어질 때, 패킷의 헤더에 도착 End System의 IP가 기록된다. 주소와 Forwarding Table를 참조하여 Router는 경로를 분석하고 다음 Router로 패킷을 Forward한다. 해당 Table은 인터넷의 여러 Routing Protocol을 통해 설정된다.

### 1.3.2. Circuit Switching

> 전화기 사례 : Call이 오면 두 전화기 사이의 경로와 자원을 예약하고(Call Set-Up and Resource Reservation), 이후에 데이터가 전달된다.

End System들간의 통신 세션이 유지되는 동안, 버퍼와 링크(일정 대역폭, 즉 전송 속도) 등 경로를 따라 데이터를 전송하는데 필요한 자원들이 먼저 예약된다. 송신자와 수신자간의 개별적인 연결(dedicated end-to-end connection)에 대한 경로 확보가 선행되어야 하며, 전송 중에는 이러한 연결 상태가 계속 유지되어야 한다. 이러한 연결을 Circuit이라고 한다.

네트워크 링크를 확보한다는 것은 데이터의 일정한 전송 속도 확보를 의미한다. 따라서, Circuit Switching은 일정한 전송 속도를 보장할 수 있다.

#### Multiplexing in Circuit-Switched Networks

링크의 Circuit은 다음 두 가지 방식 중 하나를 채택한다.

* FDM(Frequency-Division Multiplexing)
  * 링크의 주파수 스펙트럼을 발생한 연결들로 나눈다.
  * 연결이 지속되는 동안 네트워크는 일정 대역폭(Bandwidth)을 해당 연결에 할당한다.
* TDM(Time-Division Multiplexing)
  * 시간이 고정 지속시간을 가진 여러 프레임들로 나뉘고, 각각의 프레임은 고정된 수의 시간 슬롯으로 구분된다.
  * 네트워크가 연결되면 각 프레임들이 가진 하나의 시간 슬롯을 해당 연결에 할당한다.

#### Packet Switching vs Circuit Switching

Packet Switching은 딜레이로 인한 예측불가능한 변수들로 인해 실시간 서비스에 적합하지 않다는 비판을 받는다. 하지만 Circuit Switching에 비해 더 간단하고 효율적이며 비용이 저렴하다.

Circuit Switching은 데이터가 흐르지 않을 때에도 점유된 네트워크 자원들은 예약된 상태라 계속 잉여 상태에 머무른다. 때문에 다른 연결 요청들은 해당 잉여 자원들을 사용할 수 없다는 단점이 있다. 예약은 결국 네트워크 자원의 분할을 의미하며, 패킷을 송수신할 때 링크 전송 속도의 Full-Capacity를 사용할 수 없다.

반면 Packet Switching은 요청에 따라 그 때 그 때 링크를 점유하고 Full-Capacity로 패킷을 전송한다. 두 방식 모두 많이 쓰이지만, Packet Switching이 인터넷 데이터 전송에 있어서 Resource Sharing이 더 우수하다. 같은 자원으로 더 많은 유저 요청을 처리할 수 있기 때문이다.

### 1.3.3. A Network of Networks

End System은 Access Network를 통해 인터넷에 연결된다. 두 Host끼리 패킷을 송수신하기 위해 직접 연결하게 된다면, 지구상에 N개의 Host가 있을 때 시간 복잡도는 O(N^2)이 된다.

경제 논리에 의거하여 특정 지역의 인터넷을 제공하는 Regional ISP와 그 보다 더 높은 계층에서 국가 및 국제적 등 더 넓은 범위의 인터넷을 제공하는 Global(Tier 1) ISP가 생겨났다. ISP들끼리 상호 연결되어 네트워크를 형성함으로써 더 효율적인 인터넷 네트워크 구조를 가질 수 있게되었다.

![image](https://user-images.githubusercontent.com/56240505/103505657-28214380-4e9e-11eb-9d02-44b582c0d3b7.png)

즉, ISP들끼리도 더 넓은 범위의 인터넷 연결을 얻기 위해 비용을 청구-지불하는 고객-제공자의 계층 구조를 형성한다.

**PoP(Point of Presence)** : 하위계층(고객) ISP들이 상위계층(제공자) ISP로 연결할 수 있는 제공자 네트워크에 존재하는 1개 혹은 그 이상의 라우터 그룹이다.

**IXP(Internet Exchange Point)** : ISP들간의 Link를 맺어주는 제 3 사업자. IXP 없이도 ISP들 끼리 Peer Link를 맺을 수 있다.

**Multi-Home** : 하위 계층의 ISP는 여러 개의 상위 계층 ISP를 가질 수 있다.

**Content Provider Network** : Google 등 컨텐츠 제공자들은 인터넷을 통해 송수신하는 데이터의 양이 방대하다. 따라서 자신들만의 데이터 센터들의 네트워크를 형성하여 인터넷 서비스를 제공한다. 이를 통해 상위 계층의 ISP 사용 비용을 절감하고 자신들의 서비스를 유연하게 제어할 수 있다.

<br>

## 1.4. Delay, Loss, and Throughput in Packet-Switched Networks

### 1.4.1. Types of Delay

#### Nodal Processing Delay

패킷의 헤더를 분석하고 어느 방향으로 이동해야할 지 결정하는 등의 처리 지연 시간이다. 패킷에 비트 레벨의 에러가 있는지 체크하는 행위 등이 포함된다. 대부분 비슷하다.

#### Queuing Delay

Router의 Buffer에 담긴 패킷들이 Outbound Link(다음 Router로 향하는 Link)로 빠져나가는 처리 속도보다, 패킷들이 유입되는 트래픽 속도가 더 빠른 경우 발생한다. Outbound Link의 처리 속도(대역폭)가 현재 트래픽에 비해 충분하지 않아, 처리해야 할 패킷들이 Buffer에 쌓이게 되는 지연 시간이다.

``L(패킷 길이) * a(평균 패킷 유입 속도) / R(Outbound Link Bandwidth, 패킷이 처리되는 속도)``

이러한 Traffic Intensity는 0.7만 되더라도 Queue의 Delay가 거의 무한정 늘어나게 된다. 이는 a가 단위 평균이기 때문이다. 평균으로 볼 때는 Traffic Intensity가 1이더라도 Delay가 발생하지 않지만, 어떤 경우에는 R보다 L*a 값이 더 많아져 Delay가 급증하기 때문이다.

Delay가 길어져 Buffer가 꽉 차게 되면 새로 유입되는 패킷의 정보가 유실된다. 이는 이전 노드에서 재전송해야하기 때문에 리소스 낭비로 이어지고, 딜레이가 더 길어지는 원인이 된다.

#### Transmission Delay

패킷을 Output Link로 밀어넣는데 걸리는 시간을 의미한다. 패킷의 길이와 링크의 Bandwidth에 영향을 받는다. (L/R)

#### Propagation Delay

Node에서 다음 Node로 패킷이 링크를 따라 전파되는 시간을 의미한다. 즉, 물리적인 링크의 길이를 해당 링크 매체의 전파 속도로 나눈 값이다. (d/s)

### 1.4.2. Throughput

단위 시간당 Source Host부터 Destination Host에게 전달된 데이터의 양을 의미한다. 패킷 교환은 패킷 일부가 아닌 완전한 1개 패킷 전체를 받은 뒤 이루어지기 때문에, Throughput은 두 End System 사이의 여러 Link들 중 Bandwidth(Capacity 전송 속도)가 가장 작은 Link에 결정된다.

해당 Link를 병목 링크라고 한다. 주로 Network Edge 쪽에서 병목 현상이 자주 발생한다.

<br>

## 1.5. Protocol Layers and Their Service Models

### 1.5.1. Layered Architecture

프로토콜이란 방대하고 복잡한 시스템이기 때문에, 각 부분을 정의하고 다른 부분들과의 관계를 짓는 모듈화를 하는 것이 유지보수 등이 편리하다. 그러나 상위 기능과 하위 기능들간의 기능들이 중복이 되거나, 한 레이어의 기능이 다른 레이어의 정보를 필요로 하는 등의 이유로 오버헤드가 발생할 수 있다.

* 예 : Transport Layer에서 TCP를 통해 신뢰성 있는 데이터 전송을 한다고 해도 데이터 무결성이 중요한 Application은 Application Layer에서 또 데이터를 검증하는 등 유효성 검사가 여러번 발생한다.

### 1.5.2. Internet Protocol Stack

계층적인 구조로 이루어져 있으며, 각 계층별로 목적을 가지고 있다. 각 계층은 하위 계층 프로토콜의 서비스를 이용해서 자신의 목적을 달성한다

각 계층은 PDU(Protocol Data Unit)를 만들어내며, 하위 계층에 해당 PDU를 전달하며 캡슐화를 한다. 이 때 하위 계층은 상위 계층에서 전달받은 PDU(Payload)에 Header를 붙여 새로운 PDU를 생성한다.

#### Application Layer

* 네트워크 어플리케이션을 지원하는 레이어로 HTTP, FTP, SMTP 등이 있다.
* 유저 어플리케이션에서 메시지를 생성한다.

#### Transport Layer

* Source Process로부터 Destination Process에게 데이터를 전달해주는 레이어이다.
  * 네트워크는 엄밀하게 말하면 두 Host가 아닌 두 Host의 Process 끼리 통신하는 것이다.
* UDP, TCP 등이 있다.
* Application 레이어에서 내려준 메시지에 Header를 붙여서 Segment를 만든다.

#### Network

* IP와 Routing Protocol을 통해 Source Host로부터 Destination Host까지의 라우팅을 담당한다.
* 즉, 목표까지 도달하기 위해 여러 Hop을 이동하는 길찾기 기능을 한다.
* Transport 레이어에서 내려준 Segment에 Header를 붙여서 Datagram을 만든다.

#### Link

* 이웃하는 네트워크 노드들간(Hop, 경로의 한 부분) 데이터를 전달해주는 레이어이다.
* 하나의 Hop을 이동한다.
* Network 레이어에서 내려준 Datagram에 Header를 붙여서 Frame을 만든다.

#### Physical

* 비트 정보를 회선과 같은 물리적인 매체를 통해 전달하는 레이어이다.

End System, Switch, Router 등의 노드가 가지는 Protocol Stack은 각각 다르다. Switch나 Router는 굳이 Application과 Transport Layer의 정보를 확인할 필요가 없다.

노드의 네트워크 레이어들은 하위 계층부터 순차적으로 전달받은 Peer-PDU의 헤더를 분석하며, 올바른 경우 헤더를 분리한 뒤 Payload를 상위 계층으로 전달한다.

다음 Router로 이동해야 할지, 현재 위치가 Destination이기 때문에 더이상 움직이지 않고 Message를 확인하는지 등의 결정은 헤더 분석을 통해 이루어진다.

다음 노드로 이동해야 한다면, 현재 노드의 PDU는 다시 상위 계층에서 하위 계층으로 전달되면서 헤더가 붙는다. 재조립된 패킷은 이후 링크를 통해 다음 노드 위치로 이동한다.

<br>

## 1.6. Network Security

인터넷이 상용화되면서 점차 정보 보안에 대한 관심이 커졌다.

* Virus
  * 파일을 받아 실행할 경우 자가 복제하며 네트워크를 감염시킨다.
* Worm
  * 파일을 받기만 해도(즉, 유저의 명시적 실행이 없어도) 자동으로 자가 복제하며 네트워크를 감염시킨다.
* Spyware Malware
  * 웹 사이트 방문 내역, 키보드 로그 등을 기록할 수 있다.
* Botnet
  * 스팸 이메일이나 DDoS 등의 공격에 사용되는 좀비 컴퓨터들을 의미한다.
* DDoS
  * 타깃 주변의 컴퓨터들을 감염시킨 뒤, 타깃에게 쓸모없는 과도한 트래픽의 패킷을 보냄으로써 서버 및 대역폭 등 타깃의 리소스를 사용 불가 상태로 만드는 행위이다.
* Sniffing
  * WiFi 등 공유되는 브로드캐스트 네트워크에서 발생한다.
  * 제 3자가 자신이 Destination이 아닌 패킷 데이터를 받아 정보를 읽는다.
* IP Spoofing
  * 특정 Host가 다른 Host의 IP를 도용해 Source Host인척 다른 Destination Host에 Packet을 보내는 것이다.

<br>

## 1.7. Internet History

* 인터넷 아키텍쳐의 근간.
  * 미니멀리즘과 자율성.
  * 최선 노력 서비스(Best-Effort Service Model).
  * 상태 정보가 없는 라우터.
  * 분산 제어.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
