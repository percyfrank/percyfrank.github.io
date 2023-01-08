---
title: "[Effective Java] Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라"
excerpt: "인터페이스를 상수 공개용 수단으로 사용하지 말자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-27
last_modified_at: 2021-01-27
---

## 1. 인터페이스의 용도

인터페이스란 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할이다. 클래스가 어떤 인터페이스를 구현하는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기해주는 것이다

<br>

## 2. 안티 패턴 : 상수 인터페이스

> PhysicalConstants.java

```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

일부 클래스들은 외부 인터페이스에 상수를 선언해두고 가져다 사용한다. 클래스가 사용하는 상수들은 엄연히 내부 구현이기 때문에 상수 인터페이스를 사용하는 것은 API를 노출하는 행동과 다름없다. 클라이언트를 혼란시키며 오히려 클라이언트 코드가 해당 상수들을 의존하게 될 수도 있다. 특히 final이 아닌 클래스가 해당 인터페이스를 구현하면 모든 하위 클래스의 이름 공간이 해당 인터페이스가 정의한 상수들로 오염된다.

상수를 공개할 목적이라면 다음 사항들을 고려하라.

* 특정 클래스(인터페이스)와 강하게 연관된 상수는 해당 클래스(인터페이스) 자체에 추가한다.
* 열거 타입을 사용한다.
* 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
