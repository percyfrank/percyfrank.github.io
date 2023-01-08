---
title:  "OSI 모델, TCP/ICP 모델의 정의 & 존재 이유" 
excerpt: "OSI 7계층, TCP/IP 모델"

categories:
  - Network
tags:
  - Network

date: 2022-11-17
last_modified_at: 2022-11-17

---

# 1. OSI 모델

- 컴퓨터에서 데이터를 전송하기 위해서 컴퓨터 내부에서 하는 여러 가지 일들을 7 계층으로 나눈 것
- 통신이 일어나는 과정을 단계별로 알 수 있기 때문에 특정한 단계에서 이상이 생기면 그 곳만 수정하면 되기 때문에 편리하다
- 데이터를 보낼 때는 7 → 1, 받을 때는 1 → 7

    ![C2F18D62-0AD5-4B26-8C44-38F734E7B337](https://user-images.githubusercontent.com/85394884/206527254-d3101e34-cfd5-4269-9c1b-2d04bc3a5a69.jpeg)


💡 면접 질문

**OSI 7계층에 대해 설명해주세요.**

  ![Untitled](https://user-images.githubusercontent.com/85394884/206530566-02bbe967-2034-48a9-b9d0-23b5feb513ee.png)

<br>

# TCP/IP 모델

- TCP/IP는 **통신 규칙의 모음**이며, TCP/IP 모델은 이러한 규칙이나 프로토콜이 적용되는 모델
- TCP/IP 모델은 OSI 모델을 바탕으로 실생활에 사용하기 위해 만들어졌다
- OSI모델과 다르게 4계층 (표현, 세션 계층 → 응용 계층)

    ![Untitled 1](https://user-images.githubusercontent.com/85394884/206527266-516aaab8-60ed-4e8f-8642-4ec070032aea.png)
