---
title: "Blocking vs Non-Blocking & Sync vs Async"
excerpt: "작업에 대한 제어권과 순서 및 결과의 관심사로 구분할 수 있다."
categories:
  - Operating System
tags:
  - Operating System
date: 2021-06-08
last_modified_at: 2021-06-08
---

## 1. Blocking vs Non-Blocking

* Blocking이란 자신의 작업을 진행하다가 다른 주체의 작업이 시작되면 다른 작업이 끝날 때까지 기다렸다가 자신의 작업을 시작한다.
* Non-Blocking이란 다른 주체의 작업에 관련없이 자신의 작업을 하는 것이다.
* 다른 주체가 작업할 때 자신의 제어권이 있는지 없는지로 볼 수 있다.
* 기술적으로 **함수 호출** 관련해서 사용하는 개념이다.
  * A 함수를 호출했을 때, A 함수를 호출 했을 때 기대하는 행위를 모두 끝마칠때까지 기다렸다가 리턴되면 블로킹 되었다고 한다.
  * A 함수를 호출했을 때, A 함수를 호출 했을 때 기대하는 어떤 행위를  요청하고 바로 리턴되면 논블럭킹 되었다고 한다.

<br>

## 2. Sync vs Async

![image](https://user-images.githubusercontent.com/56240505/132941784-474bbef8-1461-4ab3-8f1f-3e35757524a6.png)

* 동기/비동기는 행위에 대한 개념으로, 기술적으로 구분하기가 쉽지 않다.
* Synchronous(동기)란 작업을 동시에 수행하거나, 동시에 끝나거나, 끝나는 동시에 시작함을 의미한다.
  * 호출된 함수의 수행 결과 및 종료를 호출한 함수가(호출된 함수뿐 아니라 호출한 함수도 함께) 신경쓰는 경우다.
* Asynchronous(비동기)란 두 주체가 서로의 시작 및 종료 시간과는 관계 없이 별도의 수행 시작/종료시간을 가지고 있을 때를 의미한다.
  * 호출된 함수의 수행 결과 및 종료를 호출된 함수 혼자 직접 신경 쓰고 처리하는 경우다.
* 결과를 돌려주었을 때 순서와 결과(처리)에 관심이 있는지 아닌지로 판단할 수 있다.

<br>

## 3. 조합

1. Blocking/Sync
  * Java에서 입력 메서드를 사용할 때 제어권이 넘어가, 그 뒤 코드가 실행되지 않는다.
  * 입력을 마무리 하면 제어권과 결과를 받아 그 다음 코드 라인들이 처리된다.
2. Non-Blocking/Sync
  * Blocking/Sync와 큰 차이가 없다.
3. Blocking/Async
  * 자신의 작업에 대한 제어권이 없으나, 결과를 바로 처리하지 않아도 된다.
4. Non-Blocking/Async
  * Js에서 ``fetch()``로 API 요청을 보내고 다른 작업을 하다가, 콜백을 통해 응답에 대한 추가적인 작업을 처리할 때 등이 대표적인 예시다.

<br>

---

## References

* [[10분 테코톡] 🐰 멍토의 Blocking vs Non-Blocking, Sync vs Async](https://www.youtube.com/watch?v=oEIoqGd-Sns)
* [동기/비동기와 블로킹/논블로킹](https://deveric.tistory.com/99)
