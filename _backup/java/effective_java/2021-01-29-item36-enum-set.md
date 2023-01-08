---
title: "[Effective Java] Item 36. 비트 필드 대신 EnumSet을 사용하라"
excerpt: "EnumSet 클래스는 비트 필드 수준의 성능과 명료함을 제공한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. 비트 필드

열거한 값들이 단독이 아닌 집합으로 사용되는 경우, 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

> Text.java

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; //1
    public static final int STYLE_ITALIC = 1 << 1; //2
    public static final int STYLE_H1 = 1 << 2; //4

    public void applyStyles(int styles) { ... }
}
```

> Main.java

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

* 비트별 OR를 사용해서 여러 상수를 하나의 집합으로 모은다.
  * 이러한 집합을 비트 필드라고 한다.
* 비트 필드를 사용하면 합집합과 교집합 등 집합 연산을 효율적으로 수행할 수 있다.
* 정수 열거 상수와 단점이 동일하지만, 출력을 하면 정수 열거 상수를 출력할 때 보다 해석하기 어렵다.
* 최대 몇 비트가 필요한지 API를 작성 시 미리 예측해서 적절한 타입(int or long)을 선택해야 한다.
  * API를 수정하지 않고는 비트 수를 수정할 수 없다.

<br>

## 2. EnumSet

EnumSet 클래스는 열거 타입의 상수의 값으로 구성된 집합을 효과적으로 표현해준다. Set 인터페이스를 완벽히 구현하며 타입 안전할 뿐만 아니라 다른 Set 구현체와도 함께 사용이 가능하다.

* 내부 구현은 비트 벡터로 구현되어 있다.
  * 원소가 64개 이하라면 대부분은 EnumSet 전체를 long 변수 하나로 표현한다.
* 원소들을 순회하는 대량 작업은 산술 연산을 써서 구현했다.
* 비트 필드 성능에 버금가지만 비트를 직접 다룰 때의 오류들에서 자유롭다.

> Text.java

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styles) { ... }
}
```

> Main.java

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

``applyStyles()`` 메서드는 Set 인터페이스를 인자로 받아 다양한 Set 구현체를 지원하도록 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
