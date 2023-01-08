---
title: "[Effective Java] Item 73. 추상화 수준에 맞는 예외를 던지라"
excerpt: "예외 번역 및 예외 연쇄를 사용하면 근본 원인을 분석하기 좋다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 예외 번역

수행하는 일과 관련 없어 보이는 예외가 발생하는 경우가 있다. 저수준 예외를 처리하지 않고 바깥으로 전파할 때 주로 발생한다. 이는 내부 구현 방식을 드러내어 윗 레벨 API를 오염시킨다. 다음 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램에 영향을 끼칠 수 있다.

> Translation.java

```java
try {
    ... //저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
    //추상화 수준에 맞게 번역한다.
    throw new HigherLevelException(cause);
}
```

* 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 하며, 이를 예외 번역이라 한다.
* 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는게 좋다.
  * 문제의 근본 원인을 고수준 예외에 실어 보낸다.
  * 언제든 접근자를 통해 저수준 예외를 꺼내볼 수 있다.

> HigherLevelException.java

```java
class HigherLevelException extends Exception() {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}
```

* 고수준 예외는 생성자를 통해 상위 클래스의 생성자에 원인을 건네주어, 최종적으로는 Throwable 생성자까지 건네진다.
  * ``initCause()``로 원인을 직접 못박을 수 있으며, ``getCause()``로 원인을 프로그램에서 접근할 수 있다.

<br>

## 2. 차선책

예외를 전파하는 것보다 예외 번역이 우수하지만 남용해서는 곤란하다. 가능한 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서 예외가 발생하지 않도록 한다.

* 상위 계층 메서드의 매개변수 값을 하위 계층으로 보내기 전에 유효성 검사를 진행한다.
* 아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 해당 예외를 조용히 처리하여 API 호출자에게 예외를 전파하지 않는다.
  * 로깅 기능을 활용해 미리 기록해두면, 추후 로그 분석을 통해 추가 조치를 취할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
