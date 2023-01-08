---
title: "[C] File Descriptor 및 System Call"
excerpt: "프로그램은 어떻게 파일을 열고 읽는 것일까?"
categories:
  - C
tags:
  - C
toc: true
toc_sticky: true
last_modified_at: 2020-09-20T20:13:00-05:00
---

## 1. File Descriptor

파일 디스크립터란 리눅스 및 유닉스 계열의 시스템에서 프로세스가 파일을 다룰 때 사용하는 개념으로, 프로세스에서 특정 파일에 접근할 때 사용되는 추상적인 값이다. 프로세스에서 열린 파일의 목록을 관리하는 테이블의 인덱스이다.

표준 입력(0), 표준 출력(1), 표준 에러(2)는 기본적으로 할당되어 있다. 프로세스가 실행 중 파일을 ``open()``하면, 커널은 해당 프로세스의 파일 디스크립터 숫자들 중 사용하지 않는 가장 작은 값을 할당해준다.

프로세스가 열려있는 파일에 ``read()`` 등 시스템 콜을 이용해 접근할 때, FD 값을 통해서 특정 파일을 지칭할 수 있다.

![image](https://user-images.githubusercontent.com/56240505/95673524-223ffd00-0be4-11eb-9d70-ed247a50218e.png)

FD가 3번이라는 것은, FD 테이블의 3번 항목이 가르키고 있는 파일이라는 의미이다. FD 테이블의 각 항목은 FD 플래그와 파일 테이블의 포인터를 가지고 있다. FD를 통해 포인터를 사용하여 시스템의 파일을 참조할 수 있게 된다.

<br>

## 2. System Call

시스템 콜은 UNIX/LINUX와 같은 OS(Operating System)의 Kernel에 서비스를 요청할 때에 호출하는 함수를 말한다.

![image](https://user-images.githubusercontent.com/56240505/95673584-de012c80-0be4-11eb-83ab-77b9b2e44c23.png)

OS는 커널 모드와 사용자 모드로 나뉜다. 프로그램이 구동될 때, 파일을 읽고 쓰거나 화면에 출력하는 등 많은 행위가 커널 모드를 사용한다. 시스템 콜은 사용자 모드가 커널 영역의 기능을 사용하게 해준다. 즉, 프로세스가 하드웨어에 직접 접근하여 필요한 기능을 사용할 수 있게 해준다.

<br>

---

## References

* [File Descriptor (파일 디스크립터)](https://dev-ahn.tistory.com/96)
* [System Call 함수와 Library Call 함수의 차이](https://www.it-note.kr/3)
* [운영체제 04 : 시스템 콜 (시스템 호출, System Call)](https://luckyyowu.tistory.com/133)
