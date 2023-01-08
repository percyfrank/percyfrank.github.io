---
title: "[A Top-Down Approach] CH 6. Wireless and Mobile Networks"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 6장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-12T16:55:00-05:00
---

## 6.1. Introduction

Wireless는 Link가 Wireless인 것을 의미하고, Mobility는 사용자가 움직여다니는 것을 의미한다. 비슷하게 겹치는 구석이 있으나 동등한 개념은 아니다.

<br>

## 6.2. Wireless Links and Network Characteristics

Wireless Link는 라디오 시그널을 사용해 통신한다. 라디오 시그널은 전파될 때 장애물에 부딪히며 반사되기 때문에 도착 시간이 다르기도 하며 패킷 손실이 발생하면서 시그널 세기가 약해진다. 무선 네트워크 주파수를 동시에 여러 기기들이 사용함에 따라 간섭이 일어나기도 한다.

WiFi 또한 Wired Shared Link처럼 CSMA를 통해 하나의 노드가 현재 진행중인 전송과 충돌을 일으키지 않도록 하려고 한다.

> **CSMA** : 매체를 사용하려고 시도하기 이전에 매체를 상태를 감지하여 충돌의 기회를 줄이는 것이다. 충돌의 가능성을 줄일 수 있으나 완전히 없앨 수는 없다.

그러나 충돌이 발생했을 때 Wired Link는 CD(Collision Detection) 등을 통해 문제를 극복할 수 있다. 반면 Wireless는 Hidden Terminal 및 Fading 이슈로 인해 이러한 충돌 감지가 어렵다.

또한 Wireless Link는 Wired Link보다 자원이 적어 충돌로 인한 재전송 때문에 큰 오버헤드가 발생할 수 있다. 따라서 Wireless Link는 간섭을 원천적으로 배제하는 Collision Avoidance라는 옵셔널한 기능을 사용할 수 있다.

Collision Avoidance란 Sender가 데이터 프레임을 전송할 때 랜덤 엑세스가 아니라 채널을 예약함으로써 긴 데이터 프레임 전송의 충돌을 피하는 것이다.

Sender는 CSMA를 사용하여 RTS(Request-To-Send)라는 작은 패킷을 Base Station에 보낸다. RTS는 여전히 다른 RTS와 충돌할 가능성이 있으나 크기가 작기 때문에 큰 무리가 되지 않는다.

Base Station은 RTS에 대한 응답으로 CTS(Clear-To-Send)를 브로드캐스트한다. CTS를 수신한 노드들 중 RST를 보낸 Sender만이 데이터 프레임을 보내기 시작하며, 다른 스테이션들은 해당 기간동안 데이터 전송을 연기한다.

즉, 작게 예약된 패킷들을 사용해서 데이터 충돌을 피하는 방식이다.

### 6.2.1. CDMA

Cellular Network는 CDMA(Code Division Multiple Access)를 사용하기도 한다. CDMA란 모든 유저들이 같은 주파수를 공유하지만, 각 유저들마다 데이터를 인코딩할 수 있는 고유한 Chip Sequence를 부여한다. 다중 접속을 허용하며, 최소한의 간섭으로 데이터를 다중 전송할 수 있다. 다만 데이터를 인코딩 및 디코딩 하는 과정에서 시간 등 비용이 증가할 수 있다.

<br>

## 6.3. WiFi: 802.11 Wireless LANs

Wireless End System은 정적인 기기 혹은 모바일 기기이다. Base Station이란 Wired Network에 연결되어 있으며, 일정 영역에서 Wired Network와 무선 Host들 사이에서 패킷 전송을 담당한다. WiFi와 같은 Access Point부터 Cell Tower 까지 모두 BS에 해당된다.

Wireless Link란 Mobile 기기와 BS를 연결하는 역할이다. Backbone Link로도 사용되며, Multiple Access Protocol이 협력한다. 대역폭, 속도, 거리 등이 다양하다.

Wireless Network는 두 가지 모드로 작동할 수 있다.

* Infrastructure Mode.
  * BS가 모바일을 Wired Network로 연결한다.
  * 모바일 기기는 이동 중에 연결 중인 BS가 변경될 수 있다.
* Ad Hoc Mode.
  * BS가 없다.
  * 각 Node(User Host)는 자신의 시그널이 도달할 수 있는 다른 호스트까지만 연결되어 통신할 수 있다.
  * 각 Node들이 네트워크를 형성하고 그들 사이에서 라우팅이 발생한다.

이런 두 가지 모드의 네트워크는 또한 Single Hop인지 Multiple Hops인지에 따라 세분화될 수 있다.

* Single Hop.
  * Infrastructure : Node가 BS에 도달하는데 1개 홉이 걸리는 일반적인 무선 네트워크이며, BS는 더 큰 네트워크로 연결된다.
  * Non-Infrastructure : BS가 없고 더 큰 인터넷으로 연결되지 않는다. Node들간의 주종 관계가 있는 블루투스가 대표적인 예이다.
* Multiple Hops.
  * Infrastructure : Node가 BS에 도달하는데 여러 홉을 Relay해야 하는 매쉬넷 등의 네트워크이다.
    * 사람이 없는 지역에 무선 네트워크를 제공하기 위해 BS를 설치하지 않고 데이터를 사람이 많은 지역의 BS까지 릴레이되게 한다.
    * 이를 통해 인프라 설치 비용을 절감할 수 있다.
  * Non-Infrastructure : MANET과 VANET 등 유저 호스트들이 서로 데이터를 릴레이하며 전달하지만, 외부 인터넷과 연결되지 않는다.

<br>

## 6.4. Mobility Management: Principles

Router 등 Network Core가 Mobility를 지원하기 위해서는 Router는 Routing Table Exchange를 통해 모바일 노드들의 정보를 알려야한다. 테이블이 모바일 기기들의 위치를 명시하면 End System에 대한 변화는 없으나, End System이 많아지고 이동할 때 마다 Table Entry의 양이 많아지고 자주 업데이트 되는 문제가 있다.

따라서 Mobility는 Access Network 등 Network Edge에서 지원한다.

* Permanent Address.
* Home Network.
  * Home Agent.
* Visited Network.
  * Foreign Agent.
* Care-Of-Address.

모바일 기기가 움직이게 되면 Home Agent와 Foreign Agent가 협력하게 된다. 모바일 기기는 Visited Network에서 Care-Of-Address를 발급받는다.

Correspondent는 모바일 기기의 Permanent Address를 향해 통신을 시도한다. Home Agent는 Permanent Address를 위한 메시지를 캡슐화하여 Care-Of-Address로 Forward한다. Foreign Agent는 해당 메시지의 헤더를 때고 Visited Network에 있는 Permanent Address의 모바일 기기를 찾아 메시지를 전달한다.

Network Core에 모바일 기기 위치 변경 등의 정보를 노출하지 않고도 Mobility를 지원할 수 있게 된다.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
