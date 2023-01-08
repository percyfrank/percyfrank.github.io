---
title: "Public Cloud 및 네트워크 확인"
excerpt: "우아한테크코스 배포 인프라 미션 학습 내용입니다."
categories:
  - DevOps
tags:
  - DevOps
date: 2021-06-18
last_modified_at: 2021-06-18
---

## 1. Cloud

* 인터넷의 은유로서, 정확히는 인터넷을 통해 원격으로 접근할 수 있는 모든 것을 의미한다.
* 클라우드 서비스란 메일과 드라이브 등 인터넷으로 제공되는 서비스를 의미한다.

### 1.1. Cloud Computing

* 서버, 데이터베이스, 네트워킹 등 컴퓨팅 리소스를 인터넷을 통해 관리하는 것을 의미한다.

### 1.2. 왜 사용할까?

#### 일반적인 시나리오

* 일반적인 서비스는 데이터베이스 등 저장소에 있는 데이터를 서버에서 원하는 형태로 가공해 네트워크를 통해 사용자에게 전달한다.
  * 데이터를 어떻게 관리할 것인가?
  * 서버를 어떻게 관리할 것인가?
  * 네트워크를 어떻게 관리할 것인가?
* 서비스를 할 수 있는 방식은 다음과 같다.
  * 개인 PC로 서비스한다.
  * 사무실 서버로 서비스한다.
  * 데이터 센터를 활용한다.
* 위 시나리오에서는 다음과 같은 상황들을 고려하게 된다.
  * 백업은 어떻게 하지?
  * 다른 팀과 코드를 어떻게 공유하지?
  * 이중화 구성은 어떻게 하지?
  * 서버 장비 관리 및 OS 설치는 누가 하지?
  * Rack 관리를 위해 상주 인력을 두어야 하나?
  * 유휴 장비는 얼마나 두고, 배포 구성은 어떻게 하지?
  * 보안 구성은 누가 하지?
  * DDoS 대응 장비는 있나?
  * 네트워크 장비는 누가 관리하지?
  * ...

#### 관심사의 분리

* 기존의 일반적인 시나리오에서 발생하는 고민들은 Cloud 제공 업체가 한다.
* 관심사를 분리함으로써, 개발자는 **어떤 대상에게 서비스하며 사용자가 원하는 것이 무엇인지** 등 핵심 가치에 집중할 수 있다.
  * 데이터를 어떻게 얻고, 가공하고, 보관 및 처리할 것인가?
    * Real-Time Data Stream, Triggers, Object Storage 등.
  * 어떤 대상에게 서비스할 것인가?
    * 웹, 앱, API, 인증/인가 등.
  * 사용자가 원하는 것은 무엇인가?
    * 조회 조건.
    * JSON 혹은 XML 등 데이터의 형태.
    * 데이터의 주기.
    * 데이터 정합성 및 무결성 등 보장.

<br>

## 2. Network

* 네트워크를 왜 학습해야 할까?
  * 서비스는 네트워크를 통해 사용자에게 가치가 전달된다.
  * 네트워크 상의 이슈는 서비스에 심각한 문제를 발생시키며, 안정적인 서비스 운영을 위해 학습해야 한다.
* 통신망
  * 노드들과 이들 노드들을 연결하는 링크들로 구성된 하나의 시스템이다.
* 노드와 링크?
  * 노드란 IP를 가지고 통신할 수 있는 대상으로서, 인스턴스가 될 수 있으며 네트워크 장비도 될 수 있다.

### 2.1. OSI 7 계층

![image](https://user-images.githubusercontent.com/56240505/122527124-04ba1180-d056-11eb-8fd1-289aaa658f18.png)
![image](https://user-images.githubusercontent.com/56240505/122527193-11d70080-d056-11eb-9272-f807cd25b384.png)

* OSI 7 계층은 별도로 학습하자!

### 2.2. Ping Check

> Shell

```bash
ping xlffm3.github.io -c 1
PING xlffm3.github.io (185.199.110.153): 56 data bytes
64 bytes from 185.199.110.153: icmp_seq=0 ttl=51 time=60.669 ms

--- xlffm3.github.io ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 60.669/60.669/60.669/0.000 ms
```

* Ping은 ICMP 프로토콜을 사용하며 IP 정보만으로 서버에 요청이 가능한지를 확인한다.
  * IP가 신뢰성을 보장하지 않아 네트워크 등의 에러에 대처할 수 없는데, ICMP는 오류 정보 발견 및 보고 기능을 담당하는 프로토콜이다.

> Shell

```bash
traceroute google.com
traceroute to google.com (172.217.26.46), 30 hops max, 60 byte packets
 1  ec2-52-79-0-132.ap-northeast-2.compute.amazonaws.com (52.79.0.132)  0.807 ms  0.748 ms ec2-52-79-0-94.ap-northeast-2.compute.amazonaws.com (52.79.0.94)  4.504 ms
 2  100.66.8.102 (100.66.8.102)  2.706 ms 100.66.8.88 (100.66.8.88)  6.450 ms *
```

* RTT(Round Trip Time)란, 한 패킷이 왕복한 시간이다.
* 네트워크 시간은 연결 시간, 요청 시간, 응답 시간 등으로 구성된다.
* RTT가 높을 경우 어느 구간에서 오래 걸리는지 확인해야 한다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
ip-192-168-1-1.ap-north  ether   02:82:53:92:a7:84   C                     eth0
ip-192-168-1-2.ap-north  ether   02:82:53:92:a7:84   C                     eth0

ubuntu@ip-192-168-1-31:~$ ip route
default via 192.168.1.1 dev eth0 proto dhcp src 192.168.1.31 metric 100
192.168.1.0/26 dev eth0 proto kernel scope link src 192.168.1.31
192.168.1.1 dev eth0 proto dhcp scope link src 192.168.1.31 metric 100
```

* ARP(Address Resolution Protocol)란 논리적 주소인 IP를 통해 물리적 주소인 MAC을 알아내 통신을 가능하게 도와주는 프로토콜이다.
* ARP 요청을 브로드캐스팅하면 수신한 장비들 중 자신의 IP에 해당하는 장비가 응답한다.
  * 응답받은 IP, MAC, NIC 포트 정보 등을 토대로 통신을 진행한다.

### 2.3. Port Check

> Shell

```bash
$ telnet [Target Server IP] [Target Service Port]
```

* 서비스의 정상 구동 여부를 확인할 수 있다.
* 서버는 서비스에 하나의 포트번호를 오픈해두고도 많은 사용자와 연결을 맺을 수 있다.
  * [소켓 특징](http://jkkang.net/unix/netprg/chap2/net2_1.html)

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ sysctl fs.file-max
fs.file-max = 399626

ubuntu@ip-192-168-1-31:~$ ulimit -aH
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15620
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) unlimited
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15620
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

* [최대 연결 횟수에 대하여 - 이동욱](https://woowabros.github.io/experience/2018/04/17/linux-maxuserprocess-openfiles.html)

### 2.4. Port Forwarding

> Shell

```bash
## 원격지 서버에서 8080 포트로 소켓을 열기
$ sudo socket -s 8080

## iptables 를 활용하여 port forwarding 설정
## 아래의 설정은 80번 포트로 서버에 요청을 하면 서버의 8080번 포트와 연결
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

## 서버의 공인 IP 확인
$ curl wgetip.com

## 자신의 로컬에서 연결
$ telnet [서버의 공인 IP] 80

## 설정 삭제
$ sudo iptables -t nat -L --line-numbers
$ sudo iptables -t nat -D PREROUTING [라인 넘버]
```

* 80번으로 서버에 연결 요청시에 8080번으로 연결되도록 소켓을 생성한다.
* ssh 기본 포트를 22번이 아닌 다른 포트로 변경하여 연결을 시도할 수 있으나, 전체 대역에 대해 ssh 접속 허용은 위험하다.

### 2.5. HTTP Response check

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ curl -I google.com
HTTP/1.1 301 Moved Permanently
Location: http://www.google.com/
Content-Type: text/html; charset=UTF-8
Date: Tue, 06 Jul 2021 12:54:49 GMT
Expires: Thu, 05 Aug 2021 12:54:49 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 219
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
```

* HTTP 응답 코드로 네트워크 상태를 검사할 수 있다.

> Shell

```bash
ubuntu@ip-192-168-1-31:~$ sudo systemd-resolve --flush-caches
ubuntu@ip-192-168-1-31:~$ nslookup google.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	google.com
Address: 172.217.31.142
Name:	google.com
Address: 2404:6800:4004:81f::200e

ubuntu@ip-192-168-1-31:~$ sudo tcpdump -nni eth0 port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```

* Google로 요청하는 흐름에 대하여?
  * 로컬의 DNS Cache를 확인해본다.
  * /etc/hosts 파일에 정적으로 설정한 정보를 확인한다.
  * /etc/resolv.conf에 설정한 정보를 기반으로 DNS 서버에게 질의한다.
  * DNS 서버는 정보가 있으면 반환하고 없으면 본인의 상위 DNS에게 질의를 하여 정보를 알아온다.
  * 도메인에 해당하는 IP를 알게되면 DNS Cache에 추가한다.

<br>

---

## References

* 우아한테크코스
