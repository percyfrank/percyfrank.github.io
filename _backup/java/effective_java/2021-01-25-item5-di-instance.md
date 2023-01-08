---
title: "[Effective Java] Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라"
excerpt: "의존성 주입을 통해 변화에 유연하게 대처할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 정적 유틸리티 클래스 및 싱글턴의 한계

많은 클래스가 하나 이상의 자원(의존 객체)를 필요로 한다. 맞춤법을 검사하는 클래스를 정적 유틸리티 클래스 혹은 싱글턴으로 구현해본다.

> SpellChecker.java (정적 유틸리티 클래스)

```java
public class SpellChecker {
    private static final Lexicon dictionary = new Lexicon();

    private SpellChecker() {
    }

    public static boolean isValid(String word) {
        //...
    }
}
```

> SpellChecker.java (싱글턴)

```java
public class SpellChecker {
    public static final SpellChecker INSTANCE = new SpellChecker();

    private final Lexicon dictionary = new Lexicon();

    private SpellChecker() {
    }

    public boolean isValid(String word) {
        //...
    }
}
```

위 방식들은 하나의 사전만을 사용한다. 요구 사항의 변경으로 인해 다른 종류의 사전을 사용해야 하는 경우 대처하기 어렵다. Lexicon의 final 키워드를 제거하여 setter를 통해 임의로 변경할 수 있으나, 불변성이 사라짐으로써 멀티 스레드 환경에서는 안정적이지 못하다.

> SpellChecker.java (DI)

```java
public class SpellChecker {

    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }

    public boolean isValid(String word) {
        //...
    }
}
```

인스턴스를 생성할 때 필요한 자원(의존 객체, Lexicon)을 클래스가 자체 생성하는 것이 아니라 주입해준다. 인스턴스의 불변성을 보장해주며 다양한 종류의 자원(의존 객체)을 사용할 수 있게 된다. 이를 통해 클래스의 유연성, 재사용성, 테스트 용이성이 높아진다.

<br>

## 2. 자원 팩토리

생성자에 자원 팩토리를 넘겨주는 방식을 적용할 수 있다. 즉, 팩토리 메서드 패턴을 구현하는 것이다.

* 팩토리?
  * 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체이다.
  * Java 8의 ``Supplier<T>`` 인터페이스.

``House create(Supplier<? extends Brick> brickFactory)``

팩토리가 제공하는 Brick(벽돌)들로 House(집)을 만드는 메서드이다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
