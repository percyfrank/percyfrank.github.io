---
title: "[Effective Java] Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라"
excerpt: "열거 타입 상수와 관련된 값은 인스턴스 필드에 저장한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. ordinal 메서드

``ordinal()`` 메서드는 해당 상수가 열거 타입에서 몇 번째 상수인지 인덱스를 제공한다. 그러나 열거 타입 상수와 관련된 값은 ``ordinal()`` 메서드로 얻게하지 말아야 한다.

> Ensemble.java

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, DOUBLE_QUARTET,
    NONET, DECTET, TRIPLE_QUARTET;

    public int numberOfMusicians() {
        return this.ordinal() + 1;
    }
}
```

* 상수 선언 순서를 변경하는 순간 코드가 오작동한다.
* 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
  * 이미 8중주 상수가 있기 때문에 똑같이 8명이 연주하는 관련 상수를 추가할 수 없다.
* 값을 중간에 비워둘 수도 없다.
  * 13명이 연주하는 상수를 추가하려면 중간에 12명짜리 더미 상수도 추가해야 한다.
* 전반적으로 유지보수가 어려워진다.

> Ensemble.java

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }
    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

열거 타입 상수와 관련된 값은 인스턴스 필드에 저장해야 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
