---
title: "[Effective Java] Item 72. 표준 예외를 사용하라"
excerpt: "재사용성이 좋은 표준 예외를 사용하면 다양한 이점을 누릴 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 표준 예외

Java 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다. 표준 예외를 재사용하면 우리의 API가 다른 사람이 익히고 사용하기 쉬워진다. 표준화된 규약을 준수하기 때문이다. 또한 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

* IllegalArgumentException, IllegalStateException 등.
* Exception, RuntimeException, Error, Throwable은 직접 재사용하지 말자.
  * 이 클래스들은 추상 클래스로 생각해야 한다.
  * 여러 예외들을 포괄하기 때문에 안정적으로 테스트할 수 없다.
* 더 많은 예외를 제공하길 원한다면 표준 예외를 확장해도 된다.
  * 단, 예외는 직렬화할 수 있으며 직렬화에는 많은 부담이 따르니 커스텀 예외를 너무 많이 생성하지 않는다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
