---
title: "[A Top-Down Approach] CH 5. Link Layer"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 5장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-11T16:55:00-05:00
---

## 5.4. Switched Local Area Networks

### 5.4.1. Link-Layer Addressing and ARP

32bit IP 주소는 Network Layer에서 Host와 Router 및 Link 사이의 인터페이스로서 사용되었다. Link Layer에서 사용하는 Frame 패킷 헤더에는 MAC 주소 필드가 존재하며, 이를 통해 네트워크 기기를 식별한다.

MAC(LAN, Physical, Ethernet) 주소란 Link Layer상의 하나의 인터페이스에서 물리적으로 연결된 다른 인터페이스로 Frame이 이동할 때 사용된다. 이더넷과 와이파이를 포함한 대부분의 네트워크 인터페이스에 고유하게 할당되어 있다. 48bit MAC 주소는 기기 생산시 NIC ROM에 구워진다.

IP는 논리적인 주소이고 MAC 주소는 물리적인 주소이다. Network Layer는 어떻게 특정 MAC 주소가 Datagram이 가고자 하는 IP 주소의 네트워크 인터페이스인지 식별할 수 있을까?

ARP(Address Resolution Protocol)라는 프로토콜은 ARP Table을 제공하며, LAN 상의 호스트와 라우터 등 노드들은 ARP Table을 가지고 있다. ARP Table은 노드들의 IP와 MAC 주소간의 맵핑 정보와 해당 맵핑 정보가 어느 시간 유지되는지 TTL 등을 제공한다.

A 노드는 B 노드의 MAC 주소를 알아야 Frame을 전달할 수 있다. 만약 모른다면 A는 ARP 쿼리 패킷에 B의 IP 주소를 담아 브로드캐스트한다. LAN 상의 모든 노드들이 ARP 쿼리를 받으며, B는 자신의 IP를 식별하여 MAC 주소와 함께 응답을 보낸다. A는 B의 MAC 주소로 데이터를 전송할 수 있게 된다.

주로 20분 정도 IP-MAC 주소를 ARP 테이블에 캐싱해둔다. 어드민이 ARP 테이블 작성 등에 개입할 필요 없이 노드가 스스로 브로드캐스팅을 통해 ARP 테이블을 제작할 수 있다. (plug-and-play).

네트워크 통신을 위해 Routing과 ARP는 협업을 한다. 노드는 IP 계층에서 목적지 Host IP뿐만 아니라 라우팅을 통해 경로상 다음 Hop에 위치한 노드의 IP가 무엇인지 알게 된다. Link(Ethernet) 계층에서는 경로상 다음 Hop에 위치한 노드의 MAC 주소를 ARP 브로드캐스팅을 통해 알아낸다. 이후 Datagram에 MAC 주소 등 헤더를 달아 Frame을 만든다. Frame은 1개 Hop을 이동하고 다음 노드에서도 이러한 작업을 반복한다.

다음 노드의 IP, MAC 주소를 식별하고 1개 HOP을 이동하는 작업을 반복하다 보면 패킷은 마침내 목적지 Host에 도달하게 된다.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
