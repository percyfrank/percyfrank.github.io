---
title: "[A Top-Down Approach] CH 4. Network Layer"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 4장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-07T16:55:00-05:00
---

## 4.1. Introduction

Network Layer는 Host의 Transport Layer가 내려준 Segment를 Datagram으로 캡슐화하고 Receiver Host에게 전달해주는 역할을 한다. Receiver는 반대로 Datagram의 헤데를 때고 Segment로 디캡슐화하여 Transport Layer로 올려준다.

Network Layer 프로토콜은 Host와 Router 모두에게 탑재되지만, Host의 네트워크 프로토콜은 라우팅 기능이 약하다. Subnet 외부로 나가기 위한 Edge Router 혹은 Gateway 정도를 찾아주는 제한적인 기능이다. Network Core의 Router가 라우팅 루트를 계산하는 등 더 강력한 기능을 제공한다.

### 4.1.1 Forwarding and Routing

Routing은 Source Host에서 Destination Host까지 패킷이 전달되는데 거쳐야 하는 경로를 알고리즘을 통해 계산하고, Forwarding Table을 세팅하는 것이다. 모든 Router들은 Forwarding Table을 가지고 있다. 반면 Forwarding은 실제 라우터에서 패킷을 받았을 때 헤더와 Forwarding Table을 참조하여 알맞는 Output Link로 패킷을 보내주는 것이다.

비유하자면 Routing은 전체 여행 계획을 짜는 것이고 Forwarding은 여행을 다니면서 만나게 되는 여러 갈래의 교차로에서 표지판을 보고 알맞은 길을 택해 통과하는 것이다.

### 4.1.2. Network Service Models

Datagram을 전달하는 채널을 위한 서비스 모델은 크게 두 가지로 분류된다.

* 독립적인 개별 Datagram 위한 서비스.
  * Datagram을 특정 지연 시간 내에 전달하는 것을 보장한다.
* Datagram의 플로우를 위한 서비스.
  * Datagram의 전달 순서와 최소 Bandwidth 플로우를 보장한다.
  * 각 패킷 사이의 간격 등을 제한하는 서비스가 있다.

Network는 크게 Circuit Switching과 Packet Switching으로 나뉘며, Packet Switching은 Virtual Circuit과 Datagram으로 나뉜다. Network Layer에서 Virtual Circuit은 Connection-Oriented Service를 제공하며, Datagram은 Connectionless Service를 제공한다. 이는 TCP와 UDP 관계와 비슷하다. 그러나 다음과 같은 차이점이 있다.

* TCP/UDP는 Transport Layer에서 Process간의 통신이지만, Virtual Circuit과 Datagram은 Host-To-Host 서비스를 의미한다.
* TCP/UDP는 Network Edge의 Process간의 상호작용이라면, Virtual Circuit과 Datagram은 Network Core에 있는 모든 것들이 협력해서 구현하는 결과이다.
* 유저는 TCP나 UDP를 선택할 수 있으나, Virtual Circuit과 Datagram 사이의 Network Service를 선택할 수 없다.
  * ISP가 제공하는 개념이기 때문이다.

<br>

## 4.2. Virtual Circuit and Datagram Networks

### 4.2.1. Virtual-Circuit Networks

Virtual Circuit은 데이터 전송 전후로 연결에 대한 Call Setup과 Tear Down이 수반된다.

* Call Setup을 위한 메시지가 목적지에게 도달하면서 Hop이 어디부터 어디까지인지 등 VC 경로가 결정된다.
* Call Setup시 VC에 해당하는 아이디가 부여되는데, 전송되는 패킷들이 해당 아이디를 헤더에 달고 있다.
* Packet Switching에 속하지만 Bandwidth 및 버퍼와 같은 Link와 Router 자원들이 VC에 예약(Predictable and Dedicated)될 수 있다.
* VC 경로에 위치한 Router들은 자신들을 통과하는 연결에 대해 상태 정보를 유지하고 있어야 한다.

VC별로 경로와 VC 번호(ID)를 할당할 때, 해당 정보들을 Forwarding Table에 저장한다. 해당 VC에 속한 패킷들은 도착지 IP가 아닌 VC 번호를 통해 Link와 Router를 이동한다. 이 때 VC 번호는 A부터 B까지 가는 모든 경로의 링크들이 동일하지 않고, 경로에서 링크가 바뀜에 따라(Router를 만나 갈아탐에 따라) 서로 다른 VC 번호들이 부여된다.

예를 들어, Call Setup시 경로 상의 N번째 Router는 이전의 N-1번째 Router에게 "내게 Datagram을 보낼 때 VC 번호 K를 붙여라"는 식으로 설정된다.

1개 VC에 속한 모든 Link에 대해 VC 번호를 통일하게 하면, Link별로 고유 VC 번호를 할당하는 것 보다 결과적으로 더 많은 VC 번호를 필요로 하기 때문이다. 각 Link에는 복수개의 VC가 지나가게 되는데, 1개의 Connection에 대해 경로의 모든 링크를 1개 VC 번호로 통일하게 되면 다른 VC 번호들과 겹치지 않는 값을 할당해야 한다. 할당하려고 하는 VC 번호를 다른 링크가 사용하는지 확인하는 등 컨트롤 오버헤드가 발생한다.

그러나 VC에 속하는 Link들이 서로 다른 VC 번호를 가지고 있다면? Router는 1번 Link에서 들어온 12번 VC와 2번 링크에서 들어온 12번 VC는 서로 다른 것으로 인지할 수 있게 되며, 이는 결과적으로 bit를 덜 사용하게 된다.

VC는 Router를 통해 추가적인 연결 상태 정보를 Forwarding Table에 저장할 수 있다. signaling protocols을 통해 연결에 대한 Setup, 유지, Tear Down 작업을 수행한다. ATM 등 전화 네트워크 PSN에서 사용하지만, 오늘날 인터넷에서는 쓰이지 않는다.

### 4.2.2. Datagram Networks

별도의 Call Setup이 없으며 Network 계층은 UDP처럼 Datagram을 그냥 전송하는데 최선을 다한다. Router들은 목적지 Host 주소를 보고 패킷을 포워딩한다.

Forwarding Table에는 목적지 IP 주소로 가기 위해 현재 Router에서 어느 Output Link를 사용해야 하는지 등에 대한 정보가 정의되어 있다. Router는 Datagram의 헤더에 있는 목적지 Host IP 주소를 보고 Forwarding Table을 참조하여 Datagram을 알맞은 Output Link로 보내준다.

현재 인터넷 네트워크에는 너무 많은 IP들이 존재하기 때문에, 각 IP별로 라우팅에 필요한 모든 맵핑 정보를 Forwarding에 정의할 수 없다. Forwarding Table이 너무 비대해지면 Table에서 원하는 Output Link 맵핑 정보를 Look-Up 하는데에도 많은 시간이 걸린다.

이에 대한 해결책으로 Table Entry들을 Aggregate한다. Router는 자신과 가까이 있는 Local 주소들에 대해서는 Forwarding Table에 맵핑 경로를 구체적이고 개별적으로 명시한다. 그러나 멀리 위치한 Local 주소들에 대해서는 개별적으로 맵핑 경로를 명시하지 않고, 해당 지역으로 향할 수 있는 경로로 뭉뚱그려 정의한다.

예를 들어 고속도로를 타고 서울에서 경상도 XX시를 가는 중이라고 할 때, 서울 인근의 고속도로 표지판에는 경상도 XX시, YY시, ZZ시 등 구적인 경로가 나와있지 않다. 다소 러프하게 전라도는 어느 방향, 경상도는 어느 방향 등 뭉뚱그려 설명되어 있다. 경상도 방향에 가까워질 수록 XX시, YY시, ZZ시 등 구체적인 경로 정보가 표지판에 나타난다.

### 4.2.3. Origins of VC and Datagram Networks

전화기 등이 VC를 사용하며 인터넷은 Datagram을 사용한다. 인터넷의 경우, End System가 대게 컴퓨터이기 때문에 성능이 우수하여 복잡한 작업의 처리가 가능하다. 그렇기 때문에 복잡한 작업은 Network Edge 쪽으로 몰아주고, Network Layer 등은 데이터를 전달하는 것에만 집중할 수 있도록 구조를 단순화하는데 초점을 맞추었다. 또한 인터넷을 사용하는 Application 중에는 유연한 서비스를 제공하는 것들이 많아 엄격한 Timing 준수 등 오버헤드와 관련된 제약 사항이 필요없는 경우가 많다. 아울러 Network를 연결하는 Link들의 타입이 다양해서 속도 등 특성도 천차만별이다. 그러다 보니 Datagram을 사용하는 Network의 속도가 매우 우수하다.

반면 VC의 경우 전화와 같은 End System은 복잡한 작업을 처리할 수 없다. 따라서 여러 복잡한 작업들이 Network Layer에서 진행되며 Application 자체도 대화와 관련된 것들이 많아 엄격한 Timing과 Reliablity 등 요구사항이 많다. 따라서 성능 측면에서 Datagram에 비해 우수하지 못하다.

<br>

## 4.3. What’s Inside a Router?

Management Control Plane은 프로토콜을 통해 Routing을 계산하는 등 Software의 영역에 가깝다면, 실제 데이터를 Forwarding하는 Data Control Plane은 Hardware 영역이다. Router의 구조는 크게 3가지로 분류된다.

* Input Ports.
* Switching Fabric.
* Output Ports.

### 4.3.1. Input Processing

Input Port 내부는 크게 3가지로 구분된다. 패킷을 Physical - Link - Network Layer 순으로 헤더를 떼서 Datagram으로 만든다.

* Input Port Function(Line Termination).
  * 전달받은 패킷을 bit 단위로 인식하며, bit를 모아서 Frame을 형성한다.
* Link Layer Protocol(Data Link Processing).
  * Ethernet 등.
  * Frame이 1 Hop을 잘 건너왔는지 등 Link Layer의 프로토콜을 수행한다.
* Decentralized Switching(Lookup, Fowarding, Queuing).
  * 제대로 된 Frame이라면 Forwarding Table을 참조하여 경로를 찾고 Queue에서 대기한다.
  * 이상적인 목표는 Link Transmission Rate(Bandwidth)로 들어오는 패킷들을 병목 없이 처리하는 것이다.

Router로 유입된 Datagram이 Input Port의 Queue 버퍼에서 대기하고 있으면, Switching Fabric은 Datagram을 뽑아 적합한 Output Link로 옮겨준다. Switching Fabric의 처리 속도가 느리다면 Input Port의 Queue 버퍼가 쌓이게 되면서 딜레이 및 패킷 손실이 발생한다.

특히 Switching Fabric은 2개 이상의 패킷을 동일한 Output Link로 동시에 옮기지 못한다. 이 때, HOL(Head Of Line) 블락킹이 발생하게 된다. 2개의 Input Port가 있고 각각의 Queue 버퍼에서 대기 중인 선두의 Datagram들이 동일한 Output Link로 나가기 위해 경쟁한다고 가정해보자. 경쟁에 패배한 Datagram은 다른 Datagram이 Output Link로 옮겨지는 동안 대기해야 한다. 경쟁에 패배한 Datagram은 Queue의 선두에 그대로 있기 때문에, 그 뒤에 위치한 Datagram들은 자신이 가고자 하는 Output Link가 유휴상태임에도 최선두 Datagram 때문에 처리되지 못하고 대기해야 한다.

### 4.3.2. Switching

Input으로 들어온 Datagram을 Output Link로 전달해주는 Switching Rate는 N Inputs / Output Line Rate이다. Switching Fabric에서 Switching 작업이 발생하게 된다. Switching 작업은 크게 3가지로 구현될 수 있다.

* Switching via Memory.
  * 1세대 방식으로, 고전적인 컴퓨터의 CPU 통제 하에 Switching이 발생한다.
  * Input Port와 Output Port 사이에 메모리가 있고, 패킷이 메모리로 복사된 다음 적합한 Output Port로 이동한다.
  * 즉, 1개의 Datagram이 2번의 System Bus를 타게 된다. (Bus Crossing)
    * 메모리의 Bandwidth에 따라 Switching 속도가 제한된다.
* Switching via Bus.
  * Input Port와 Output Port를 독점적으로 연결하는 Bus 구조를 사용한다.
  * Datagram을 Input Port 메모리에서 Output Port Memory로 이동하는데 공용 버스를 사용한다.
  * Switching 속도는 Bus의 Bandwidth에 따라 결정된다.
    * 각 Connection들이 서로 Bus에 타려고 경쟁하기 때문이다.
* Switching via Interconnection Network
  * Single Bus 방식의 대역폭 제한을 극복하기 위해 고안되었다.
  * 크로스바 방식 등은 멀티 프로세서들간의 연결을 위해 개발되었다.
  * 더 빠른 성능을 위해 Datagram을 고정 길이의 Cell로 쪼개서 이용한다.

### 4.3.3. Output Processing

Input Port 구조와 동일하지만 순서만 다르다. Input Port가 Physical - Link - Network Layer 순으로 패킷의 헤더를 때고 Datagram으로 만들면 Switching Fabric에서 길을 찾는다. 이후 Output Port에서 Network - Link - Physical Layer 순으로 Datagram에 Header를 붙여 전송함으로써 Link를 물리적으로 1 Hop씩 이동하게 된다.

Queue에 쌓인 Datagram을 전송하는 방식에는 다양한 스케쥴링 규칙이 있다. 똑같이 Buffering으로 인한 패킷 손실이 발생할 수 있다.

### 4.3.4. Where Does Queueing Occur?

권장되는 Buffering Capacity 양은 RTT * C(Link Capacity) / N Square(파이프를 지나는 플로우의 개수)이다.

<br>

## 4.4. The Internet Protocol (IP): Forwarding and Addressing in the Internet

인터넷의 Network Layer에는 Routing 프로토콜 외에도 IP 프로토콜과 ICMP 프로토콜이 있다. Routing 프로토콜이 Management Control Plane이라면, IP 프로토콜은 Data Control Plane이다.

### 4.4.1. Datagram Format

Datagram의 너비는 32bit(4byte)이며, 헤더는 고정적인 파트가 총 5개이다. 또한 가변길이의 필드를 가질 수 있다. 헤더에는 다음과 같은 필드가 있다.

* Protocol 버전 넘버.
* 가변길이 헤더의 길이.
* Data 타입.
* TTL(Time To Live) : Hop을 건널 수 있는 최대 시간이며 고스트 데이터를 방지한다.
* Upper Layer : Payload에 실린 Segment가 UDP인지 TCP 인지 명시한다.
* Flag, Offset, Checksum 등.

옵션 필드에 경유 경로를 확인하는 등의 정보 추가 기능이 있다. IP Datagram 크기는 고정 크기(20byte의 TCP 고정 헤더와 20byte의 IP 고정 헤더) 및 그 외 Application Layer 오버헤드의 합이다.

#### IP Datagram Fragmentation

Link Layer의 Frame이 전달할 수 있는 가장 큰 데이터의 양을 MTU(Maximum Transmission Unit)이라고 한다. Link 타입마다 MTU가 다르다.

크기가 큰 IP Datagram은 Link Layer에서 하나의 Frame으로 캡슐화가 될 수 없기 때문에, Network Layer에서 MTU 정보를 가지고 Router가 하나의 Datagram을 더 작은 Datagram으로 쪼갠다. Datagram의 파편들은 각각 독립적인 Datagram으로서 운송된다. 잘라진 Datagram들은 다음 Link에서 재조립되지 않고, 최종 목적지에서 재조립된다.

Fragmentation 이슈는 Network Layer에서 불가피하게 처리할 수 밖에 없다. 그러나 복잡한 기능은 최대한 Network Edge에 위치시키는 것이 Network의 철학이기 때문에, 재조립은 Network Edge에서 관장하게 된다. Link마다 분리와 조립이 반복되면 오버헤드가 커진다.

목적지의 Network Layer는 IP Datagram의 Header를 참고하여 파편화된 Datagram들을 조립하고, Segment 메시지를 추출하여 상위 계층으로 전달한다.

### 4.4.2. IPv4 Addressing

IP 주소란 Host와 Router 인터페이스를 위한 32비트의 주소이다. 인터페이스란 Host와 Router간, 그리고 물리적인 Link 사이의 연결(문)을 의미한다. Router는 여러 인터페이스를 가지고, Host는 한 개 혹은 두 개의 인터페이스(WiFi, Ethernet)을 가진다.

예를 들어, 1개 Router는 3개의 Host를 향해 각각 다른 IP 인터페이스를 가지고 소통할 수 있다. Host는 하나의 IP 인터페이스를 가지고 다른 노드들과 소통한다. 일반적으로 8비트씩 4파트를 점으로 구분하여 나눈다.

IP 주소는 Subnet 파트와 Host 파트로 나뉜다. Subnet 파트는 IP 주소 중 앞 부분에 해당하며 Host 파트는 뒷 부분에 해당한다. Subnet이란 IP 주소들 중 동일한 Subnet 파트를 가진 인터페이스 디바이스들의 집합을 의미한다. Subnet에 속한 디바이스들은 Router를 거치지 않고 서로에게 물리적으로 도달할 수 있다.

xxx.xxx.xxx.xxx/24와 같이 마지막에 Subnet Mask라는 bit를 명시하여 IP 주소 중 어느 부분까지가 Subnet 파트인지 명시할 수 있다. 일반적으로 ISP는 CIDR(Classless Interdomain Routing)을 통해 일련의 주소 블럭을 서브넷으로 할당한다. 주소는 a.b.c.d/x의 형식이며, 각 클래스에 따라 수용할 수 있는 호스트의 개수가 다르다.

#### Obtaining a Host Address: the Dynamic Host Configuration Protocol

> Hierarchy : DHCP - UDP - IP - Ethernet - Physical

IP 주소는 크게 시스템 관리자가 시스템 파일에 하드코딩하거나, DHCP(Dynamic Host Configuration Protocol)를 통해 할당되어진다.

DHCP는 Application Layer의 프로토콜이며, 서버로부터 호스트가 접속할 때마다 IP를 동적으로 할당받아 사용하고, 접속을 종료하면 다시 회수한다. 이를 통해 다른 Host에게도 해당 IP를 할당이 가능하다. 특히 모바일 유저는 항상 특정 Subnet에 속해있지 않기 때문에 고정 IP보다 동적인 IP가 자원을 덜 낭비한다. 주소 재사용을 통해, 궁극적으로 주어진 IP 주소 개수보다 더 많은 수의 호스트들에게 서비스를 제공할 수 있다.

End System이 Subnet 영역에서 시스템일 키게 되면, DHCP 클라이언트 프로그램이 자동으로 실행되면서 해당 Subnet 지역을  관리하는 DHCP 서버와 통신하게 된다. 일련의 요청 - 응답 과정을 통해 인터넷 접속에 필요한 IP 주소를 할당받게 된다. DHCP는 IP 할당 이외에도, 클라이언트의 First-Hop Router과 DNS 서버의 주소 및 IP 주소의 Subnet 파트를 알려준다.

IP 주소 체계 또한 계층적이다. Forwarding Table은 Router와 목적지 IP까지의 모든 맵핑 정보를 개별적으로 정의하지 않는다. 멀리 위치한 IP들에 대한 경로는 뭉뜽그려서 해당 IP들이 속해있는 특정 기관이나 Subnet으로 향하는 맵핑 정보만을 Aggregate한다. 이를 통해 Forwarding Table에서 Routing 엔트리 정보의 양을 줄일 수 있다.

Router는 자신과 가까운 목적지일 수록 더 긴 Prefix의 Forwarding Entry를 가지며, 멀 수록 더 짧은 Preifx Forwarding Entry를 가진다. "A.A.A로 시작하는 주소는 내 쪽으로 포워딩하라''는 Forwarding 조건은 그 뒤에 ``or``로 "B.B.B.B 주소도 내 쪽으로 포워딩해라" 등의 추가 조건을 추가할 수 있다. 인터넷 라우팅은 여러 Forwarding 조건들이 중복되는 경우 가능한 가장 상세하고 긴 Prefix에 해당하는 조건을 선택해 Forwarding한다.

인터넷진흥원 같은 한국 TLD 서버는 .kr 도메인 관리와 더불어 기관들에게 Subnet 주소 할당 등의 역할도 한다.

#### Network Address Translation (NAT)

CIDR은 클래스 기반의 IP 할당 전략이지만, 어떤 클래스는 가용할 Host의 개수가 부족하고 어떤 클래스는 남아도는 등 자원을 효율적으로 사용하지 못했다. IP 사용자가 폭발적으로 늘어나면서 효율적인 사용의 중요성이 증대되었다.

NAT는 Subnet의 각 End System들에게 IP를 할당하는 것이 아니라, Subnet 자체에게 하나의 IP를 주는 것이다. Subnet에 속한 End System들은 Private IP를 사용하지만, 외부 Subnet과 Datagram을 주고 받을 때에는 모두 하나의 NAT IP 주소를 통해 소통한다. 다만 End System별로 포트 번호가 다르다.

* 원리.
  * Subnet의 First Edge Router는 다른 Subnet과 소통할 수 있다.
  * Private IP 및 포트 번호를 가진 Subnet 내부 Host의 Datagram이 외부로 나갈 때, First Edge Router는 Datagram의 Source 주소를 NAT IP 주소와 새로운 포트 번호로 변경한다.
  * 이후 외부에서 NAT IP를 Destination 주소로 하는 Datagram이 First Edge Router로 들어온다.
  * First Edge Router는 포트 번호와 Translation Table을 참고하여 NAT IP로 이루어진 Destination 주소를 실제 Host의 Private IP 주소로 변경하고 Datagram을 전달한다.

NAT IP를 사용하면 Subnet에 속한 Local Host들은 일련의 IP 블록을 받을 필요 없이, 단 하나의 IP만 받으면 된다. 특히 Private IP들은 외부에 Subnet 외부로 노출되지 않기 때문에 보안 측면에서 우수하며, 자유롭게 변경이 가능하다. 또한 기기들의 주소 변경 없이 ISP를 변경할 수 있다.

효율적으로 IP 자원을 사용할 수 있지만, Network Layer인 Router가 포트 번호같은 Transport Layer까지 건들이기 때문에 논란이 다소 많다. 또한 NAT Traversal 문제가 있다. 서버가 NAT IP를 사용하는 Subnet에 속해 있으면 Private이며, 외부에 있는 클라이언트는 해당 서버를 향한 NAT IP를 먼저 알 수 없다. Subnet 내부의 서버가 먼저 외부의 클라이언트에 접촉을 시도해야 Private IP - NAT IP간의 Translation Table 정보가 라우터에 저장되기 때문이다.

이에 대한 솔루션으로 테이블을 수동 생성하는 것이다. 정적으로 특정 포트 번호로 온 연결 요청을 서버로 Forwarding하도록 설정하는 것이며, UPNP를 통해 이를 자동화할 수 있다. P2P 방식은 NAT 뒤에 숨겨진 서버 문제를 관리하기 어렵다.

### 4.4.3. Internet Control Message Protocol (ICMP)

Host와 Router들이 Network Level 정보를 전달할 때 사용한다. Source Host에게 어떤 에러가 있었는지 리포팅하거나 요청과 응답에 대한 Ping을 Echo하여 서로의 상태를 체크한다.

ICMP는 IP(Network Layer) 위에 존재하며, ICMP 메시지는 TCP 등의 Segment처럼 IP Datagram 내부로 캡슐화된다.

TTL(Time To Live)이란 패킷이 Router에서 폐기되기 전 까지 네트워크에 존재할 수 있는 시간, 혹은 홉을 의미한다. Traceroute 같은 서비스는 ICMP 메시지에 TTL을 활용하여 패킷의 경로와 목적지 도착 여부 등을 확인한다.

### 4.4.4. IPv6

기술의 발전으로 인해 IP 할당 수요는 폭발적으로 증가함에 따라 할당가능한 32bit인 IPv4의 개수는 고갈되고 있다. CIDR, DHCP, NAT를 통해 IP 자원을 효율적으로 사용하더라도 IP 주소가 부족하다. IPv6는 길이가 더 긴 128bit의 주소 체계를 정의하여 주소 고갈 문제를 해결할 수 있다.

과거에는 Link 쪽의 Transmission 속도로 인해 병목 현상이 많이 발생했으나, 네트워크의 물리적인 기술이 발달하면서 현재는 Router의 Switching 부분에서 병목 현상이 주로 발생한다. Router의 Forwarding 최적화를 위해 IPv6에서는 가변 길이가 아닌 40byte 고정 길이의 헤더 포맷을 사용하도록 디자인되었다. 또한 Fragmentation을 허용하지 않으며, 패킷의 크기를 Link가 처리하지 못하는 경우에 대한 예외 코드가 추가되었다.

QoS(Quality of Service) 헤더를 지원하는데, 이를 통해 플로우에 있는 Datagram의 우선 순위를 설정하고 데이터 전송에 특정 수준의 성능을 보장하려고 한다. 미래에는 멀티미디어 등이 중요하기 때문이다.

IPv6 Datagram의 Header 포맷에는 IPv4에 없는 다음과 같은 필드들이 존재한다.

* Priority : 플로우의 Datagram의 우선 순위에 대한 필드이다.
* Flow Label : 같은 플로우의 Datagram을 식별하는 필드이다.
* Next Header : Optional 헤더가 있는지, 아니면 다음에 오는 헤더가 UDP인지 TCP인지 등을 구분한다.

Checksum 등의 필드가 삭제되었으며 IPv6는 멀티캐스트 그룹 관리 기능을 제공한다.

모든 Network 시스템들이 동시에 IPv6로 업그레이드되는 것은 불가능하다. 점진적으로 IPv4를 사용하는 시스템들이 IPv6로 업그레이드가 되는데, 불가피하게 IPv4와 IPv6가 공존하는 과도기를 겪게 된다. IPv4를 사용하는 시스템은 IPv6 메시지를 이해할 수 없기 때문에 Tunneling을 사용하게 된다.

IPv6를 사용하는 라우터들끼리 통신할 때 IPv4 라우터를 거쳐야한다면, IPv6 Datagram을 Payload로 하고 IPv4의 Header를 붙여 IPv4가 이해할 수 있는 Datagram으로 캡슐화한다. IPv4 라우터는 해당 Datagram을 정상적으로 처리할 수 있게 되며, IPv6 라우터는 전달받은 IPv4 Datagram을 IPv6 Datagram으로 디캡슐화하여 사용하게 된다.

<br>

## 4.5. Routing Algorithms

라우팅 알고리즘은 패킷이 Source에서 Destination에 도달할 수 있도록 경로를 분석하고 Forwarding Table을 세팅하는 것이다.

Graph Abstraction을 사용하는데, Router Set(Vortex)과 Link Set(Edge)로 Graph 객체를 만든다. Link의 Bandwidth와 Congestion 혹은 사용료에 따라서 Cost 가중치가 다르게 설정된다. 라우팅 알고리즘은 이런 가중치 등을 분석해서 최소 비용의 경로를 발견하는 것이다.

라우팅 알고리즘은 크게 두 가지 방식으로 분류될 수 있다.

* Global : LS(Link State) 방식이 대표적이다.
  * 각 라우터는 네트워크 전체 정보를 가지고 있으며, 항상 모든 라우터가 일관되게 전체 정보를 유지해야 한다.
* Decentralized : DV(Distance Vector) 방식이 대표적이다.
  * 각 라우터는 물리적으로 가까우며 연결된 이웃들에 대한 지엽적인 비용 정보만을 안다.
  * 이웃들과 "나는 XX 서브넷에 갈 수 있고 비용은 YY야" 등의 정보를 교환한다.

다른 분류 기준도 존재한다.

* Static
  * Link의 길이와 같은 비용은 거의 변함없기 때문에, Forwarding Table이 자주 변하지 않는다.
  * 시간에 따라 천천히 변화해도 되기 때문에 라우트 알고리즘 업데이트 주기가 긴 편이다.
* Dynamic
  * 라우트 알고리즘 업데이트가 주기적으로 진행되며, Forwarding Table 변화가 빠르다.
  * 라우팅 성능을 위해 Link의 비용 변화를 체크하고 이를 지속적으로 Forwarding Table과 알고리즘에 반영하는 것이다.
  * Static과 비교했을때 동적이고 빈번하게 라우팅 정보들이 업데이트 되기 때문에, 오버헤드가 많이 발생할 수 있다.
  * 그러나 업데이트 주기를 너무 길게 잡으면 최신 비용 정보가 반영되지 못해 의도했던 성능 상 이점이 떨어지게 된다.

Dynamic 방식의 비용은 트래픽의 양과 같다. 어느 특정 링크에 발생하는 트래픽이 많으면 병목현상 등 비용이 높아지기 때문에 다른 링크를 찾으려고 라우팅을 업데이트한다. 트래픽 상황에 기반한 알고리즘은 노드의 비용이 계속 변경되기 때문에 Dynamic하게 업데이트되면서 테이블과 목적지에 대한 경로 또한 계속 변경된다.

TCP 등 데이터 전달 순서가 중요한 경우, 데이터가 전달되는 과정에서 계속 경로가 변경됨에 따라 Out Of Order가 발생할 수 있다. Receiver는 ACK 재전송 등을 해야하기 때문에 Application Layer 성능에 영향을 줄 수 있다.

또한 데이터가 전달되는 과정에서 테이블이 변경될 때, 변경 내용이 미처 모두에게 퍼지지 못해 잘못된 정보의 경로를 타게 되는 경우 라우팅 루프에 빠질 수 있다(Oscillation).

### 4.5.1. The Link-State (LS) Routing Algorithm

Dijkstra 알고리즘을 사용한다. 하나의 노드는 네트워크의 다른 모든 N개의 노드들과 정보를 브로드캐스트하며 최단 경로를 찾는다. N번의 반복을 통해 Node Tree Graph를 그리게 되고, 하나의 노드에서 다른 노드와의 최단 거리를 구할 수 있다.

그래프에서 경로 탐색을 위한 비교는 최대 N * (N+1) / 2만큼 수행될 수 있기 때문에 O(N^2) 시간 복잡도를 가지나, 효율적으로 구현하면 O(NlogN)이 된다.

### 4.5.2. The Distance-Vector (DV) Routing Algorithm

Bellman-Ford Equation을 사용하며, X 노드에서 Y 노드까지의 최소 비용 경로는 X 노드에서 X 노드의 이웃으로 가는 비용 + 이웃에서 Y까지 가는 비용의 합들 중 최소값이다.

DV를 위해 각 노드는 이웃들과의 비용을 알고 있고, 이를 이웃들에게 업데이트 한다. 노드가 본인의 Local Link 비용을 변경하거나, 이웃으로부터 비용 변경 메시지를 받을 때 알고리즘(혹은 테이블)이 업데이트된다. 업데이트가 되면 노드는 다시 이웃들간의 거리를 재계산하고, Distance Vector(다른 노드까지의 최단 거리)들에 대해 이웃들에게 노티한다.

DV의 단점으로는 좋은 정보(비용이 줄어드는 경우)는 반영이 빠르지만, 나쁜 정보는 반영이 느리다. 비용이 늘어나는 경우 각 노드들은 DV를 계산하는데 시간이 오래 걸려, 전송 중이던 패킷들이 목적지에 도달하지 못하고 무한 루프(count to infinity)에 빠질 수 있다.

Poison Reverse로 이러한 문제를 어느정도 해결할 수 있으나 3개 이상의 노드가 관여되어 있을 때 등 예외 케이스들은 해결할 수 없다.

#### A Comparison of LS and DV Routing Algorithms

* Message Complexity
  * LS : N개의 노드와 E개의 링크가 있으면 O(N^E) 메시지를 보낸다.
  * DV : 근접한 이웃들간의 메시지 교환이 발생한다.
* Convergence Speed : 정보 변경으로 인한 라우팅 업데이트.
  * LS : O(N^2) 알고리즘으로 인해 메시지는 O(N^E)만큼이 필요하다.
  * DV : Convergence는 다양하다. 좋은 정보는 준수하지만 최악의 경우 무한 루프에 빠질 수 있다.
* Robustness : 라우터가 맛이 간다면?
  * LS : 노드는 잘못된 링크 비용을 홍보하고 각 노드는 자기의 테이블만을 계산한다.
    * 문제가 발생한 부분 근처의 경로는 잘못되겠지만, 나머지 부분은 문제가 발생하지 않는다.
  * DV : 노드는 모든 잘못된 경로 비용을 홍보한다.
    * 한 노드의 테이블은 다른 노드의 테이블이 사용하기 때문에, 에러가 네트워크 전반으로 전파된다.

### 4.5.3 Hierarchical Routing

네트워크가 커지면 LS와 DV 방식 모두 하나의 노드가 가져야하는 정보의 양이 많아진다. 실제로 전체 네트워크를 하나의 네트워크 알고리즘으로 돌리지 않는데, 네트워크 스케일 때문에 Routing Table Exchange 규모가 너무 커지기 때문이다.

또한 각 기관들의 네트워크 어드민은 외부에 내부 네트워크 테이블을 알리지 않고 자신의 네트워크 라우팅을 컨트롤하고 싶어할 수 있다.

라우팅 또한 Subnet처럼 계층적으로 관리할 수 있다. 제한된 지역의 한 기관에 속한 Router들을 Aggregate하고 이들을 1개의 AS(Autonomous System)에 속하게 한다. 같은 AS에 속한 라우터들은 같은 라우팅 프로토콜을 사용하며, 속해있는 라우터들에 대한 라우팅 테이블을 만든다. 이를 Intra AS Routing Protocol이라고 한다. 서로 다른 AS에 속한 라우터들은 다른 라우팅 프로토콜을 사용할 수 있다.

하나의 AS Edge에서 다른 AS의 Router와 연결되는 Router를 Gateway Router라고 한다. 하나의 AS에서 다른 AS 내부의 목적지와 통신하기 위해 사용되는 것이 Inter AS Routing Protocol이다. 서로 다른 AS의 Gateway Router들은 Inter AS Routing Protocol을 통해 다른 AS를 사용하면 어떤 목적지에 도달할수 있는지 등의 정보를 교환한다. Inter AS Routing Protocol은 또한 해당 정보를 AS 내의 노드들에게 공유하는 역할을 한다. 이러한 협력을 통해 AS들간의 라우팅을 위한 Forwarding Table을 만들게 된다.

만약 1개의 AS에서 다른 AS의 서브넷으로 이동하는데 다양한 Gateway Router(즉, 다양한 AS)를 거칠 수 있는 경우, Inter AS Routing Protocol을 통해 라우팅을 선택하게 된다.

* Hot Potato Routing을 주로 사용하는데, 현재 Source Node 위치에서 가장 가까운 AS 내부 Gateway Router를 통해 갈 수 있는 경로를 택한다.
* 내가 속한 AS에서 리소스를 가장 덜 사용하면서 Gateway Router를 통해 빠져나갈 수 있는 경로를 선택하기 때문이다.
* 나와 거리가 먼 Gateway Router를 통해 다른 AS로 빠져나가면, 그만큼 현재 AS에서 많은 Hop을 이동하며 자원을 사용하게 된다.

<br>

## 4.6. Routing in the Internet

### 4.6.1. Intra-AS Routing in the Internet: RIP

IGP(Interior Gateway Protocol)라고도 한다.

RIP(Routing Information Protocol)은 가장 초창기에 사용한 프로콜이며, Distance Vector 라우팅 알고리즘을 사용한다.

Link 코스트를 매우 단순하게 측정하기 때문에 대부분의 링크가 비슷한 네트워크를 대상으로 한다. 링크 코스트는 대게 1이며 링크 코스트의 합은 Hop Counts가 된다. 최대 Hop Counts는 15이다.

30초마다 이웃들의 응답 메시지에 반응하며 DV 정보를 교환하는데, Advertisement 메시지에 실릴 수 있는 목적지 서브넷의 개수는 최대 25개이다. 만약 180초 동안 이웃들과 통신되지 않으면 Link Failure가 발생했다고 간주하고, 해당 이웃을 통해 갈 수 있는 경로들을 전부 무효화시킨 뒤 다시 경로를 계산해서 이웃들에게 전파한다.

DV는 Poison Reverse를 사용해도 count to infinity 무한 루프 문제를 해결할 수 없으나, RIP는 최대 경로가 15이기 때문에 무한 루프 문제에서 자유롭다.

RIP는 라우팅 프로토콜로 Application Layer의 프로세스이다. Advertisement 메시지를 만들고 UDP 패킷에 해당 정보를 싣고, Network Layer의 IP Datagram을 통해 주변과 통신한다. 이를 통해 Network Layer의 Forwarding Table이 업데이트된다.

### 4.6.2. Intra-AS Routing in the Internet: OSPF

RIP는 여러 제약사항으로 인해 대규모 라우팅에 적용하기 어렵다. OSPF(Open Shortest Path First)는 Public Availabe한 프로토콜이다. Link State 알고리즘 사용을 사용하며, OSPF 프로토콜은 IP 바로 위에서 구현되기 때문에 UDP나 TCP에 캡슐화되지 않는다.

RIP와 다르게 더 큰 규모의 네트워크에서 계층적으로 사용할 수 있으며, 모든 OSPF 메시지가 인가되기 때문에 보안상 이점이 있다. 또한 RIP는 Link 비용을 모두 1로 고정하지만, OSPF는 Delay, Bandwidth, Money 등 다양하게 계산하여 적용할 수 있다.
  * 각 Link 비용을 Advertise하기 때문에 각 Link 비용에 따른 다양한 Table(Multiple Cost Metrics)가 가능하다.
  * 멀티캐스트 라우팅(MOSPF)를 제공한다.
  * 특정 목적지까지 가는 같은 비용의 경로가 여러개인 경우 RIP는 1개만 테이블에 기록하지만, OSPF는 모두 기록한다.

**Hierarchical OSPF?**

LS는 패킷 정보가 노드 전체로 퍼져야하기 때문에, 네트워크 규모가 클 수록 패킷 전송량이 많아져서 오버헤드가 커진다. AS를 여러 개로 나누어 Inter AS Routing Protocol을 적용하듯, OSPF 또한 1개 AS를 일정한 크기의 여러 Area로 나누고 해당 지역 내부에서는 LS Broadcasting한다.

각 Area의 Border Router(Gateway Router와 유사)는 자신이 속한 Area의 내부 Router들의 그래프를 알고 이를 외부 Area의 Border Router들과 교환한다.

Area Border Router들은 Backbone이라는 지역 내에서 서로 연결되며, 해당 라우터들을 연결해주는 것은 Backbone Router들이다. Backbone 내부에서는 또한 하나의 OSPF 프로토콜(즉, LS Broadcasting)이 동작한다.

Backbone 또한 Boundary Router가 있고, 다른 외부의 Backbone의 Boundary Router들과 Backbone 내부 및 하위에 있는 경로들과 그래프를 Inter 교환하는 식으로 계층적인 OSPF를 구현한다.

### 4.6.3. Inter-AS Routing: BGP

BGP(Border Gateway Protocol)은 Inter Domain간에 사용하는 유일한 프로토콜이다. Subnet이 다른 인터넷 네트워크에게 자신의 존재를 홍보하는 방법이다.

* eBGP : 외부 AS들과 교환을 통해 Subnet 경로 정보를 가져온다.
* iBGP : 받아온 외부 Subnet 경로 정보를 AS 내부 라우터들에게 전파한다.

경로 정보 및 기타 정책 등에 기반하여 다른 네트워크에 대한 "좋은 라우터"를 구별한다.

* BGP 원리
  * 두 AS의 Gateway Router들이 Semi-Permanent TCP 커넥션을 통해 BGP 메시지를 교환한다.
    * OPEN, KEEPALIVE, UPDATE 등의 정보가 담겨있다.
    * 자신이 속한 AS 내부의 Prefix를 홍보한다.
  * 정보 교환은 향후 해당 정보로 요청이 들어오면 상대방쪽으로 Forward 해주겠다는 의미이다.

BGP 라우터들이 주고받는 핵심 정보는 다음과 같다.

* 목적지 서브넷의 Prefix + Attribute = "Route"
* Attirbute(경로 정보)의 핵심 정보는 다음 두 가지이다.
  * AS-PATH : 해당 Prefix를 가는데 거쳐야 하는 AS(예, AS68)를 의미한다.
  * NEXT-HOP : Gateway Router가 다음 Hop에 어느 AS로 가는지 등을 의미한다.
    * 두 AS간에 Multiple한 링크가 맺어졌을 수 있기 때문에, 어떠한 Gateway를 선택해야 하는지 정보가 필요하다.

Advertisement를 통지받은 Gateway Router들은 내부 중요 정책들을 기반으로 승인 혹은 거절한다.

이전 장에서도 언급했지만, 다른 AS로 갈 수 있는 Gateway Router가 여러개 인 경우 여러 정책들을 기반으로 경로를 선택하게 된다.

* 1) 보안 정책 등의 이슈.
* 2) 짧은 AS 경로.
* 3) Hot Potato Routing.
* 4) 기타 기준.

BGP는 라우팅 루프 문제가 발생하지 않는다.

#### Routing Policy

외부에서 전달받은 경로를 내부에 전파하거나, 내부 정보를 외부로 유출시킬 때 정책에 영향을 받는다.

Dual Homed와 같이 여러 네트워크들이 서로 연결되어 있는 경우, 하나의 AS가 외부에 정보를 알려주지 않아도 다른 AS들이 더 빠르게 갈 수 있는 경우 정책에 의거해 라우트 정보를 공유하지 않는다.

* Policy.
  * Intra : 싱글 어드민이 컨트롤하기 때문에 정책 결정 시스템이 필요없다.
  * Inter : 어드민이 어떻게 누가 트래픽을 라우트하는지 등 컨트롤을 원한다.
* Scale.
  * 계층적인 라우팅은 Forwarding Table 크기와 테이블의 업데이트 트래픽을 줄인다.
* Performance.
  * Intra : 성능에 초점을 둘 수 있다.
  * Inter : 성능보다 Policy가 중요할 수 있다.

<br>

## 4.7, Broadcast and Multicast Routing

4.6.장 까지는 Unicast와 관련된 내용들이었다.

Broadcast란 하나의 노드에서 여러 목적지 노드로 패킷을 전달하는 것을 말한다. 별도의 브로드캐스트 혹은 멀티캐스트 라우팅없이 유니캐스트 라우팅으로도 이를 구현할 수 있다.

```
r1 -> r2 -> r3
         -> r4
```

그러나 위와 같은 네트워크 경로가 있을 때, r1이 모든 노드들에게 데이터를 전달한다고 가정해보자. Source인 r1이 r2, r3, r4에게 모두 데이터를 전달하면, Source와 가까운 링크에서 복사가 많이 일어나며 부하가 심해진다.

목적지 개수만큼 Duplication이 발생하기 때문에 비효율적이며, Source가 네트워크에 있는 모든 목적지에 대해 다 알아야하는 문제도 있다.

### 4.7.1. Broadcast Routing Algorithms

* Flooding.
  * 노드가 브로드캐스트 패킷을 받으면 자기와 연결된 이웃들에게 모두 복사해서 보낸다.
  * 내가 예전에 브로드캐스트했던 패킷이 또 오면, 이를 또 이웃들에게 다시 뿌리는 Broadcast Storm 단점이 있다.
* Controlled Flooding.
  * 노드는 자기가 브로드캐스트하지 않은 패킷에 대해서만 브로드캐스트한다.
    * 자기가 브로드캐스트한 패킷 아이디를 일정 기간동안 외워둔다.
  * RPF(Reverse Path Forwarding).
    * 자기와 Source간의 거리가 짧은 패킷에 대해서만 브로드캐스트 포워딩을 한다.
    * Spanning Tree를 대략적으로 그리는 방식이다.
* Spanning Tree.
  * Tree에 속한 노드들만 브로드캐스트를 송수신한다.
  * 그러나 먼저 트리 그래프를 그려야한다.

Unicast는 그래프만 알면 됬었지만, Multicast는 그래프와 더불어서 그래프 어디에 멀티캐스트를 필요로 하는 호스트가 있는지를 확인해야 한다. 따라서 멀티캐스트를 위한 별도의 테이블이 필요하다.

* Source-Based Tree.
  * 멀티캐스트 호스트 소스에 따른 트리를 N개 그린다.
  * 각 소스별로 최단 경로를 확인할 수 있는 최적의 방법이지만 Shared-Tree 특성상 시간이 오래 걸린다.
* Center-Based Tree.
  * 멀티캐스트 소스들의 정보를 Center Router에게 보내고, 멀티캐스트는 센터를 소스로 하는 경로만을 사용한다.
  * 가장 최적화된 경로가 아니라는 단점이 있다.
  * 네트워크 규모가 크면 사용할 수 있다.
* Group-Shared-Tree.
  * IPv6 헤더 등에서 사용하는 Group Member 개념으로 멀티캐스트 브랜치 멤버인지 아닌지 확인한다.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
