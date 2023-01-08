---
title: "[A Top-Down Approach] CH 3. Transport Layer"
excerpt: "Computer Networking : A Top-Down Approach(James F. Kurose 저) - 3장."
categories:
  - Network
tags:
  - Network
toc: true
toc_sticky: true
last_modified_at: 2021-01-06T16:55:00-05:00
---

## 3.1. Introduction and Transport-Layer Services

Transport Layer는 서로 다른 호스트에서 작동 중인 Application 프로세스들간의 논리적인 통신을 제공한다.

* Sender 측은 Application 메시지를 Segment로 쪼개어 Network Layer에 패스한다.
* Receiver 측은 Segment들을 조립하여 메시지로 만든 다음, Application Layer로 올려 보낸다.

Internet은 TCP와 UDP를 모두 지원하며, 어플리케이션은 1개 이상의 프로토콜 사용이 가능하다.

### 3.1.1. Relationship Between Transport and Network Layers

Network Layer는 IP 주소를 통한 호스트들간의 논리적인 통신이다. 그러나 Transport Layer는 프로세스들간의 논리적인 통신이다. 이 때, Transport Layer는 Network Layer의 서비스를 개선 및 강화하여 더욱 신뢰성있게 만들어준다.

* UDP는 단순히 목적지 Host로 데이터를 전달하는데 그친다면, TCP는 패킷이 유실되거나 순서가 잘못되지 않도록 보장한다.

### 3.1.2. Overview of the Transport Layer in the Internet

Transport Layer는 Network Layer 서비스에 추가 서비스를 얹어 개선한 프로토콜 서비스를 제공함으로써, 프로세스들간의 커뮤니케이션을 지원한다.

그 중, TCP는 데이터 유실 방지 및 순서 보장 등 신뢰성있는 데이터 전달 프로토콜이다. Connection Setup, Flow Control(Receiver Buffer Overwhelming), Congestion Control(Network Overwhelming)등을 통해 실현한다.

반면 UDP는 데이터 전달 과정에서 데이터가 유실되거나 순서가 바뀔 수 있다. Host - Host 간의 IP 주소를 통한 통신(Network Layer)이 발생하면, Process 데이터 전달만 해주는 것이다. TCP는 반대로 Network Layer 서비스에 추가적인 서비스를 제공함으로써 신뢰성있는 데이터 전달이 가능하다.

두 방식 모두 딜레이 개런티나 최소 대역폭 개런티 등의 서비스를 제공하지 못한다.

<br>

## 3.2. Multiplexing and Demultiplexing

Client의 Application Layer에는 여러 프로세스들이 존재하며, 각각의 프로세스들은 소켓을 통해 메시지를 Transport Layer로 내려 보낸다.

* Multiplexing
  * 여러 소켓을 통해 내려오는 메시지를 Transport Layer에서 다루는 것을 의미한다.
  * 각 메시지에 헤더를 붙여 Segment로 만들며, 헤더에 목적지 Host 주소 등을 기록하여 Network Layer여러 소켓의 데이터를 다루며, 메시지 패킷의 헤더에 목적지 등 주소를 찍어 Network 계층으로 보내준다.
* Demultiplexing
  * 여러 호스트들에게서 받은 메시지들이 Network Layer를 통해 Transport Layer로 올라온다.
  * Transport Layer는 각 메시지들의 헤더를 분석하여 목적지 프로세스(Port)를 식별한 뒤, 해당 Port의 소켓에 메시지를 밀어넣어 Application Layer로 올려 보내준다.

### Connectionless Multiplexing and Demultiplexing

UDP의 경우 Client와 Server 모두 소켓을 1개만 보유한다. 소켓은 Application Layer에 위치한 프로세스가 Transport Layer와 소통할 수 있는 창구이며, Port 번호로 식별한다. 소켓이 생성될 때, OS가 암묵적으로 해당 Host의 프로세스의 Local Port 번호를 할당해준다.

메시지가 Transport Layer로 내려오면, UDP는 목적지의 IP와 Port 번호를 헤더에 기록한다. Host의 IP 및 Port 번호는 OS가 Segment의 헤더에 기록해준다. Sender가 이렇게 Segement를 만들어 보내면, Receiver는 Segment의 헤더에서 Dest IP를 통해 올바른 Host로 온 메시지인지 확인하고 Dest Port와 일치하는 포트 번호의 소켓에 Segment를 밀어 넣어 Application Layer로 올려 보낸다.

Demultiplexing은 Receiver에서 일어나는데, 클라이언트와 서버는 모두 다 Receiver가 될 수 있다. Dest Port 번호의 소켓으로 Segment를 Demultiplexing하는 작업은 Transport Layer에 속하지만, 해당 Segment가 올바른 Dest Host로 왔는지 IP(Network Layer) 확인이 필요하다. Port 번호는 다른 Host들도 중복해서 사용할 수 있기 때문이다.

이 때문에 IP 주소를 Segment의 Header에서 기록하고 분석하는 등 Network Layer의 서비스가 Transport Layer에서도 사용된다.

### Connection-Oriented Multiplexing and Demultiplexing

TCP를 사용하는 서버는 도어 소켓이 있고, 클라이언트의 각 요청마다 소켓을 새로 생성한다. 따라서 Demultiplexing을 위해서 Dest IP 및 Port 뿐만 아니라 Host IP 및 Host Port에 대한 정보도 필요하다. 서버가 가지고 있는 여러 소켓들 중 어떤 클라이언트 호스트의 어떤 프로세스가 요청한 커넥션인지 확인해야 하기 때문이다.

C에서 소켓 프로그래밍을 할 때 요청이 들어오면 소켓으로 데이터를 주고 받을 자식 프로세스를 fork한다. 해당 자식 프로세스는 데이터의 송수신 전달만을 담당한다. 따라서 하나의 Application 프로세스 밑에 여러 프로세스가 생기는 멀티 쓰레딩 서버가 된다.

<br>

## 3.3. Connectionless Transport: UDP

UDP는 Connectionless해서 Handshaking을 거치지 않는다. 각각의 Segment들은 서로 독립적이며, Receiver는 순서가 맞는지 모른다.

UDP는 Network Layer에서 Host들간의 길을 찾는 서비스를 통해 데이터만 충실하게 전달만 할 뿐(Best Effort), 추가적인 Transport Layer 차원의 서비스를 제공하지 않는다. 따라서 데이터 순서 변경 혹은 손실 등을 신경쓰지 않는다.

커넥션이 없어 TCP에 비해 Header가 가볍고 지연 시간이 짧으며 비용이 저렴하다. 혼잡도 관리 등의 기능이 없기 때문에 가능한 최대 속도로 데이터를 전달한다.

UDP를 사용 예는 다음과 같다.

* Streaming 멀티 미디어
  * 데이터 Loss를 사용자가 인지할 수 없을 만큼은 용인되기 때문에 Reliable이 중요하지 않다.
  * 사용자가 불편함을 느끼지 않을 만큼의 최소 프레임 등을 보장해야 하기 때문에, 최소 처리량(Throughput) 등 Rate에 민감하다.
  * TCP가 Congestion 혹은 Flow Control을 사용하여 속도를 조절하기 때문에 부적절하다.
* DNS
  * Local DNS Server는 맵핑 정보를 찾기 위해 Root 서버, TLD 서버, 기관 서버에 각각 메시지를 보낸다.
  * Iterative Query 특성상 각 서버와 1회성 메시지만 주고받기 때문에 커넥션을 맺을 필요가 없다.
* SNMP
  * 네트워크 관리 프로토콜로서, 근처의 네트워크들이 자신의 상태를 일정 주기로 관리자에게 보고한다.
  * 주기적으로 최신 정보를 업데이트하기 때문에, 데이터가 손실되어도 이후 최신 데이터로 다시 보고되는 만큼 TCP의 중요도가 덜하다.  
* 데이터 무결성(Data Integrity)이 중요한 경우.
  * Transport Layer에서 Reliable한 TCP를 사용하더라도 Application에서도 무결성 검사 등을 진행하기 때문에 두 계층 간의 기능들이 중복된다.
  * 오버헤드를 줄이기 위해 UDP를 사용하기도 한다.

### 3.3.1. UDP Segment Structure

UDP 헤더에는 Source Port와 Dest Port 및 IP 등의 정보만 있으면 충분하다. 그 외 UDP Segment의 전체 길이 및 Checksum 등의 필드가 있다.

#### UDP Checksum

Sender측은 Segment 컨텐츠를 헤더 필드까지 포함해서 16비트의 정수로 표현하고, Checksum을 계산한다. Receiver 측 또한 Segment의 Checksum을 계산하고, Sender가 계산한 Checksum과 동일한지 확인한다.

만약 동일하지 않다면 데이터 전송 중 비트 정보가 변경된 에러로 간주한다. 그러나 간단한 계산의 특성상 값이 변경되었더라도 두 Checksum이 동일한 경우가 존재하기 때문에, 모든 에러를 잡을 수 없다. 따라서 UDP는 제한적으로 데이터가 동일한지 확인하는 수준에 그치며, 그 추후의 액션은 프로토콜이 정의하지 않았다.

<br>

## 3.4. Principles of Reliable Data Transfer

Checksum 외에도 신뢰성있는 데이터 전달을 위한 여러 원칙들이 존재한다.

Receiver는 Segement를 받고 에러가 없는지, 있다면 어떤 패킷이 문제가 있으니 다시 보내달라는지 등의 Acknowlegement 응답을 보낸다. 만약 네트워크가 중간에 Drop해버려서 유실된 패킷이 있다면, Receiver는 Ack 응답을 보내지 않을 것이며 Sender는 자신이 보낸 패킷에 대한 Ack 응답을 계속 기다리게 될 것이다.

이를 방지하는 것이 Timer이다. Sender는 특정 시간동안 Ack 응답을 받지 못하면 해당 패킷을 다시 재전송한다. 그러나 Timer도 문제가 존재한다. 만약 Receiver가 패킷을 잘 받고 Ack 응답을 보냈으나, 렉 등으로 인해 Sender가 Timer 시간 내에 Ack 응답을 받지 못하고 Time-Out이 되는 경우이다. Receiver는 정상적으로 패킷을 받았지만, Sender는 재전송을 하게 된다.

따라서 Sequence Number를 패킷의 헤더에 기록해두어, Receiver는 현재 전달받은 패킷이 Segment의 몇 번째 바이트인지 확인하고 중복된 패킷인지 등을 구분할 수 있다.

Sender와 Receiver 사이에 데이터 전달을 위한 파이프가 존재한다. 1개의 패킷을 보내고 ACK 응답이 와야 다음 패킷을 보낸다면 비효율적이다. 파이프가 넓고 길다면 응답이 오고 다시 보내는데 걸리는 시간이 오래 걸리기 때문이다.

Pipelining이란 응답이 오지 않더라도 파이프에 지속적으로 데이터를 넣어주고, 반대 쪽에서 데이터를 빼주는 것을 의미한다. Window는 ACK 응답이 오지 않아도 파이프에 계속 밀어넣을 수 있는 패킷 데이터의 최대 개수이다. 해당 Window 양만큼 파이프에 데이터를 계속 넣어준다.

<br>

## 3.5. Connection-Oriented Transport: TCP

### 3.5.1. The TCP Connection

TCP는 Point-to-Point로서 Sender와 Receiver가 하나씩 존재한다. Connection-Oriented, Flow Control, Congestion Control 등을 통해 Reliable한 데이터 전송이 가능하다. 또한 Pipelining을 지원한다.

데이터의 순서를 보장하며, Byte-Stream이다. MSS(Maximum Segment Size)라는 개념이 있는데, UDP는 Application Layer에서 내려준 메시지를 해당 사이즈대로 잘라 Segment를 만들어 보낸다. 그러나 TCP는 MSS에 대한 개념이 없고 메시지들을 Byte 단위로만 인식한다. 버퍼에 메시지를 담아두고 Receiver의 Flow 상태를 봐가면서 Byte 단위로 쪼개어 Segement화 해서 보낸다.

또한 Full Duplex 서비스이다. A에서 B로 데이터 전송할 때 동시에 B에서 A 방향으로 데이터 전송이 가능하다.

### 3.5.2. TCP Segment Structure

TCP Segment는 크게 Header와 Payload로 나뉜다. Header는 고정 파트와 가변 파트로 나뉘는데, 가변적인 길이 때문에 Header 길이에 대한 필드가 존재한다. UDP는 Header가 고정이라 Segment 전체에 대한 길이 필드가 있다.

고정 Header 라인이 5파트이며 너비는 32bit(4byte)이기 때문에, TCP Segment를 만드는데 기본적으로 20byte가 필요하다.

TCP Header에는 다음의 필드들이 정의되어 있다.
* Sequence Number.
* ACK Number.
* Receive Window.
  * Receiver의 Buffer 상태를 보고 Window 및 Pipelining을 조정하여 오버플로우가 나지 않도록 Flow Control을 한다.
* Checksum.
* Flag.
  * Connection 생성, 제거, 리셋 및 ACK 관련 플래그가 정의되어 있다.

Sequence Number란 Segment에 실은 데이터의 첫 번째 바이트의 번호이다. Acknowlegement Number는 받기를 희망하는 다음 바이트 번호이다. Cumulative한 특성이 있기 때문에, ACK 번호는 해당 번호 이전 까지의 데이터는 순서대로 잘 받았다는 의미가 된다.

### 3.5.3 Round-Trip Time Estimation and Timeout

Segment를 전달하고 특정 RTT만큼 기다렸으나 ACK 응답이 오지 않는다면 패킷을 다시 보내도록 설정하는 Timer 기능이 있다. 문제는 해당 RTT를 정확하게 예측하는 것이 어렵다. 너무 작게 설정하면 데이터를 불필요하게 재전송할 것이며, 너무 길면 Segment 유실이 발생했을 때 반응이 너무 늦어지게 된다.

따라서 ACK 응답이 올 때마다 샘플 RTT들을 뽑아서 평균 RTT를 계산한다.

``EstimatedRTT = (1 - 알파) * EstimatedRTT + (알파 * 샘플 RTT)``

알파는 일반적으로 0.125로 현재 샘플이 평균에 크게 반영되지 않다는 점을 의미한다. TCP는 ACK가 처음 전송된 Segment에 대한 ACK인지 재전송된 Segment에 대한 ACK인지 구분하지 못하기 때문에, 재전송된 Segment에 대한 ACK 응답은 RTT 평균 계산에 포함하지 않는다.

그러나 해당 RTT 계산식 역시 반영률이 좋지 못해서, RTT 편차 등을 계산하여 평균 RTT를 계산한다. 만약 편차가 크다면 Timer 시간을 길게 설정할 것이다.

### 3.5.4. Reliable Data Transfer

단순히 Host(IP 주소)만 보고 찾아가는 Network 계층 위에, Pipelining과 ACK, TIMER 등의 서비스를 추가하여 TCP(Transport Layer)는 신뢰가능한 데이터 전송 서비스를 제시한다.

Sender가 붙이는 프로토콜 Header는 Receiver의 파트너 레이어가 분석한다.

#### Sender Scenarios

TCP는 Application Layer의 메시지를 Byte 단위로 인식하여 쪼개고 Segment들로 만든다. Sequence 번호는 Segment의 첫 번째 바이트의 번호이며, cumulative하게 관리된다.

Sender는 Segment를 보낼 때 Sequence 번호를 붙인다. Timer는 아직 ACK 응답을 받지 못한 가장 오래된 Segment에 대한 것이다. 현재 보내는 Segment 이전의 Segment들이 모두 ACK되었다면, Timer를 작동시킨다. 만약 타임아웃이 발생하면 해당 Segment를 재전송하며, 타이머를 재시작한다.

전송한 Segment에 대한 ACK 응답을 받았다면, Unacked였던 Segment를 ACK된 Segment로 업데이트한다. 아직 응답받지 못한 Segment가 있으면 타이머를 시작한다.

Timer를 설정할 UNACKED Segment가 더 이상 없으면 Timer를 설정하지 않는다.

```
              *92번
[---------]  [--------------------]  [--------]  [                ]
[ACK된 SEG], [SENT BUT UNACKED SEG], [NOT SENT], [할당되지 않은 부분]
```

* Sender가 Sequence 번호 92의 8byte Segment를 보낸다.
* Receiver가 Segment를 수신하고 그 다음 받고자 하는 바이트 번호 100을 ACK으로 보낸다.
* Sender는 ACK을 받으면 관리 중인 Sequence Number를 업데이트한다.

```
              *92번   *100번
[-----------------]  [------------]  [--------]  [                ]
[ACK된 SEG        ], [SENT BUT UNACKED SEG], [NOT SENT], [할당되지 않은 부분]
```

* 데이터 크기만큼 Sequence 번호를 올리고 데이터를 보낸다.
* 즉, Sender는 Sequence 번호 100의 8byte Segment를 보낸다.
  * Receiver가 이를 잘 받는다면 108 ACK 응답을 보낼 것이다.
* 새로운 UNACKED Segment가 있으면 타이머를 작동시킨다.

Receiver는 받았던 Segment들에 대해 ACK 응답을 보냈고, 현재 120번 Sequnce 번호의 데이터를 받아야하는데 Time-Out으로 인해 이전에 받은 100번 Segment를 또 받을 수 있다. 이런 경우 Receiver는 해당 데이터를 버리고, 현재 받아야 하는 120 ACK 번호를 다시 Sender에게 보낸다.

만약 Receiver가 Sequnce 번호가 각각 80과 100인 20byte Segment 두 개를 연달아 받았다고 가정해보자. 100 ACK 응답과 120 ACK 응답이 각각 Sender에게 돌아갈 것이다. 그런데 첫 번째 100 ACK 응답이 유실되고 Sender는 두 번째 120 ACK 응답만을 받는 경우가 있을 수 있다. 그러나 Sequnce 번호는 cumulative하게 관리되기 때문에, Sender는 120 ACK만 보더라도 80 Sequence 번호의 Segment가 잘 전달되었음을 판단하게 된다.

#### ACK Generation

Receiver는 올바른 순서로 도착한 Segment를 보면 ACK 응답을 바로 보내지 않고 다음 추가적인 Segment가 올 때 까지 최대 500ms 기다린다. 만약 다음 추가적인 Segment가 없으면 ACK 응답을 보내며, 있다면 해당 Segment까지 확인하고 누계된 ACK 응답을 보낸다.

TCP 헤더는 UDP에 비해 다소 비용이 비싸지만, UDP와 다르게 Pipelining을 이용하기 때문에 그 다음에 오는 Segment 또한 같은 클라이언트 및 같은 프로세스의 것이라는 것을 확신할 수 있다. 따라서 뒤에 추가적으로 오는 Segment가 있다면 해당 Segment까지 확인하여 ACK를 계산하고 응답을 1번만 보내는 것이 오버헤드 측면에서 유리하다.

만약 잘못된 순서의 Sequence 번호 Segment가 온다면, Receiver는 원래 받아야 할 순서의 ACK 응답 1개를 즉시 재전송한다. (잘못된 순서로 들어온 Segment는 저장하거나 버릴 수 있다.)

N + 100 Segment(잘못된 순서)를 먼저 받고 이후 N Sequence Segment(정상 순서)를 받는 경우가 있다. 이렇게 Gap을 채우는 Segment는 다음 Segment를 기다리는 행위 없이 바로 ACK 응답을 보낸다.

#### Fast Retransmit

Timer가 Timeout 되기까지 걸리는 시간은 상대적으로 길며, 이는 소실된 데이터를 Sender가 재전송하기까지 긴 시간이 소요됨을 의미한다.

Triple Duplicate Acknowlegement를 통해 유실된 Segment를 감지하여 Timeout을 기다리지 않고도 Segment를 신속하게 재전송할 수 있다.

Sender가 5개의 Segment를 보냈는데 2번째 Segment가 Receiver에게 가지 못하고 유실되었다고 가정해보자. Receiver는 3, 4, 5번째에 해당하는 Segment들을 잘못된 순서로 간주하고 2번째 순서의 Segment를 달라는 ACK 응답을 보낼 것이다.

Sender는 자신이 보낸 Segment를 다시 보내달라는 ACK 요청이 3번 중복되면 소실된 것으로 간주하고 Timeout까지 기다리지 않고 해당 Segment를 재전송한다. 3번이나 기다리는 이유는 불필요한 전송을 피하기 위해서이다.

### 3.5.5. Flow Control

OS는 Application과 Transport Layer를 연결하는 소켓을 생성할 때 버퍼를 할당한다. Receiver로 전달된 Segment들은 버퍼에 대기해있다가 소켓을 타고 Application Layer로 올라가게 된다. 버퍼에서 Segment들이 빠져나가는 속도보다 더 빨리 Segment들이 유입되면 Queueing Delay가 발생하며 패킷 손실 및 재전송 등 리소스 낭비로 이어진다.

TCP는 AWK 응답에 현재 Receiver의 수용 가능한 윈도우 버퍼 상태(rwnd) 등을 기록하며 통신한다. 따라서 Sender측은 Receiver의 rwnd를 고려하여 오버플로우가 나지 않도록 데이터의 흐름을 조절한다.

### 3.5.6. TCP Connection Management

TCP 연결을 셋업하는 Handshaking 과정에, Sender와 Receiver는 처음으로 보낼 Sequenc 번호 및 받기를 기대할 ACK 번호와 버퍼 사이즈 등을 서로 공유한다.

TCP는 2-Way Handshaking이 아닌 3-Way Handshaking을 적용한다.

* 2-Way Handshaking
  * 클라이언트가 서버에게 연결을 요청하면 서버는 연결을 Establish한다.
  * Establish된 서버가 클라이언트에게 응답을 보내면 클라이언트 또한 Establish된다.

클라이언트가 보낸 첫 번째 서버 연결 요청이 Timeout되는 경우, 클라이언트는 요청을 재전송한다. 두 번째 요청은 정상적으로 처리가 되어 클라이언트와 서버는 연결을 맺고 통신 작업을 수행하게 된다. 모든 작업이 마무리되면 클라이언트와 서버의 연결을 끊게 된다. 문제는 연결이 끊어진 이후에 지연되었던 처음의 요청들이 서버에 늦게 도착하는 경우이다. 더이상 작업할 필요가 없음에도 서버는 클라이언트와의 연결을 맺고 데이터를 Accept하는 등 자원 낭비가 발생한다.

* 3-Way Handshaking
  * 클라이언트가 서버에 연결을 요청한다.
  * 서버는 클라이언트의 연결 요청을 Accept한다는 응답을 보낸다.
    * 2-Way Handshaking의 경우 이 때 서버가 Establish된다.
  * 클라이언트는 서버가 Alive라는 응답을 받은 뒤 Establish 되며, 서버에게 연결을 맺었다는 응답을 보낸다.
  * 서버는 클라이언트가 Alive라는 응답을 받은 뒤 최종적으로 Establish된다.
  * SYNACK을 서로 주고 받으며 3-Way Handshaking을 적용한다.

TCP는 Full Duplexed라서 양방향의 스트림이 흐른다. TCP에서 연결을 종료하고자 할 때, Segment에 FIN bit 1을 붙여 의사를 밝힌다.

* 클라이언트는 FIN bit을 보낸다.
* 서버는 ACK 응답을 보낸다.
* 클라이언트는 더 이상 데이터를 보내지 않으며 서버가 종료되기를 기다린다.
* 서버도 이후 FIN bit을 보낸다.
* 클라이언트는 FIN을 받으면 마지막 ACK 응답을 보낸다.
  * 클라이언트는 Segment의 최대 라이프타임의 2배 정도를 기다린다.
  * 혹시라도 서버가 클라이언트의 마지막 ACK 응답을 받지 못해 종료되지 못하는 것을 대비하기 위함이다.
* 클라이언트 대기 시간이 끝나면 연결이 끊기며, 자원들이 해제된다.

양쪽의 스트림은 각각 클로징되어야 하며, 동시에 클로징이 되어도 가능하다. 대게 클라이언트가 먼저 클로징 요청을 보낸다.

<br>

## 3.6. Principles of Congestion Control

Congestion Control은 Network Layer에서 너무 많은 트래픽들로 인해 데이터가 유실되는 것을 방지한다. Router 등에서 병목 현상이 발생해 Segment 유실 및 재전송이 발생하면 Goodput(단위 시간당 처리량 중 오버헤드 비트 및 재전송 등을 제외한 것)이 줄어들게 된다.

* End-To-End Approach
  * Network Layer가 혼잡에 대한 피드백을 Transport Layer에게 주지 않는다.
  * End System에서 손실 및 지연이 발생하는 것이 관측하면 TCP가 대처한다.
* Network-Assisted Approach
  * Router가 End System들에게 피드백을 제공한다.
  * 혼잡이 있는지 싱글 비트로 알려주거나, 혹은 Sender에게 구체적인 전송 속도를 명시하는 등의 방식이 있다.

<br>

## 3.7. TCP Congestion Control

Sender는 사용가능한 Bandwidth 내에서 cwnd(Pipelining을 통해 연속적으로 보낼 수 있는 패킷 데이터의 양, 즉 Sender측의 Window)를 계속 늘려나간다.

* Additive Increase : 매 RTT마다 cwnd를 1MSS씩 늘려나간다.
* Multiplicative Decrease : Segement 손실을 감지하면 cwnd를 절반으로 줄인다.

TCP는 Additive Increase와 Multiplicative Decrease를 반복하며 혼잡을 관리한다. cwnd는 네트워크 혼잡 상황에 영향을 받기 때문에 매우 동적이다.

TCP는 cwnd 바이트만큼 Segment를 보내고 ACK 응답이 오기까지 RTT만큼을 기다린다. 따라서 TCP의 전송 속도는 대략 cwnd / RTT (bytes/sec)에 수렴한다.

### Slow Start

TCP 연결이 시작되면 초기 cwnd는 1MSS이다. 처음에는 파이프로 Segment를 1개만 보낸다. TCP는 ACK 응답이 1개 올 때 마다 cwnd를 1씩 증가시킨다. 따라서 cwnd는 매 RTT마다 2배씩 Exponential하게 증가한다.
  * 1, 2, 4, 8, 16...

### Congestion Avoidance

ssthresh라는 Threshold 값이 있으며, cwnd는 해당 값보다 낮을 땐 Slow Start를 사용하여 지수처럼 증가한다. cwnd가 ssthresh를 넘는 순간부터 Congestion Avoidance가 시작되면서 cwnd는 매 RTT마다 1씩 증가한다. (선형적이다.)

ssthresh는 OS가 세팅하는 기본값이며, 패킷 손실이 발생할 때 ssthresh 또한 cwnd / 2로 조정된다.

### Fast Recovery

ssthresh는 로스가 발생하면 cwnd / 2로 줄어든다. TCP는 크게 두 가지 방식으로 로스를 감지한다.

* Timeout을 통한 감지.
  * cwnd가 1 MSS로 조정되기 때문에 cwnd는 Slow Start처럼 매 RTT마다 2배씩 Exponential 증가하게 된다.
* TCP가 Triple Duplication Acknowlegement를 통해 Loss 감지.
  * TCP Reno 방식이다.
  * ssthresh는 cwnd / 2가 되고, cwnd도 절반으로 줄어든다.
  * 따라서 cwnd는 선형적으로 증가하게 된다.

예전 방식인 TCP Tahoe는 로스를 감지하면 무조건 cwnd를 1 MSS로 설정한다.

### Macroscopic Description of TCP Throughput

Pipelining을 통해 여러 개의 Segment들이 거의 동시에 전송되고 ACK 응답 또한 거의 동시에 들어오기 때문에, 1개의 Segment를 송수신한 것이 아니더라도 1 RTT로 간주할 수 있다.

cwnd가 K이라면 Throughput은 K / RTT가 된다. 로스가 발생했을 때의 cwnd의 평균 크기가 W일 때, TCP의 cwnd는 통계적으로 W와 W / 2 사이를 왔다갔다 한다.

따라서 TCP의 평균 cwnd 크기는 (3/4) * W가 되며, 평균적인 Throughput은 (3/4) * (W/RTT)가 된다.  

기술의 발전으로 인해 파이프가 길어지고 Bandwidth가 넓어지게 된다. TCP 최대 효율을 이끌어내려면 Window 사이즈도 커져야 한다. 그러나 여러 연구들에 따르면 TCP Throughput을 극대화하기 위해서는 Segment 손실 발생 확률이 극단적으로 낮아야한다. 이러한 문제 때문에 TCP를 계속 연구 중에 있다.

### 3.7.1. Fairness

Bottleneck이 발생하는 Router의 Link Bandwidth Capacity가 R이고, K개의 커넥션이 접근 중이다고 가정해보자. TCP가 공평하다면 각각의 커넥션은 R / K 만큼의 속도로 데이터를 전송할 것이다.

그러나 TCP는 공평하지 않고 각 커넥션들은 서로 cwnd를 늘리려고 경쟁하다 보니, R Capacity를 초과하여 병목 현상과 Segment 손실이 발생하게 된다. 각 커넥션들은 이후 Multiplicative Decrease로 인해 Throughput이 비율적으로 감소하게 된다.

이론상 2개의 커넥션이 공유하는 자원의 기울기는 1인 비례 상태가 되어야하지만, 실제로는 정반대인 반비례 상태이다.

또한 두 커넥션이 하나의 자원을 두고 경쟁을 할 때, 자원에 물리적으로 더욱 가깝게 위치한 커넥션이 먼저 빠르게 cwnd를 증가시켜 자원을 독식하기도 한다.

UDP와 Pararell TCP 연결 또한 공정함과 대치된다. 하나의 어플리케이션이 여러 TCP 연결을 맺으면, 그보다 적게 연결을 맺은 어플리케이션은 자원 경쟁에서 뒤쳐지기 때문이다.

<br>

---

## References

* Computer Networking : A Top-Down Approach(James F. Kurose 저)
* [KOCW 컴퓨터 네트워크](http://www.kocw.net/home/cview.do?mty=p&kemId=1046412)
