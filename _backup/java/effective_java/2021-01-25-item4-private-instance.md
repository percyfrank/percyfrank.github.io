---
title: "[Effective Java] Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라"
excerpt: "정적 유틸리티 클래스는 클래스의 용도를 명시하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 정적 유틸리티 클래스

정적 메서드와 정적 필드만을 담은 유틸리티 클래스는 객체지향적이지는 않지만 개발할 때 유용하게 사용할 수 있다. 그러나 정적 유틸리티 클래스는 인스턴스로 만들어 사용하기 위해 설계된 클래스가 아니다.

생성자를 명시하지 않으면 컴파일러는 자동으로 기본 생성자를 클래스에 추가해준다. 사용자는 해당 생성자가 자동 생성된 것인지 의도된 것인지 알 수 없다. 추상 클래스로 명시해도 하위 클래스를 만들어 인스턴스화할 수 있기 때문에 부족하다.

> Utility.java

```java
private Utility () {
    throw new AssertionError();
}
```

예외를 던질 필요는 없으나 해당 클래스 내부에서 실수로 인스턴스화하는 것을 방지하기 위해 퇴화시킨다. 이를 통해 해당 클래스는 인스턴스화를 할 필요가 없는 유틸리티 클래스임을 명시할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
