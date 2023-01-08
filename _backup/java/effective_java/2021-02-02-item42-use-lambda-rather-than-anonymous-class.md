---
title: "[Effective Java] Item 42. 익명 클래스보다는 람다를 사용하라"
excerpt: "람다는 작은 함수 객체를 아주 쉽게 표현할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-02
last_modified_at: 2021-02-02
---

## 1. 함수 객체

Java에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했다. 이런 인터페이스의 인스턴스를 함수 객체라고 하며, 특정 함수나 동작을 나타낸다. 함수 객체를 만드는 주요 수단은 익명 클래스였다.

> SortFourWays.java

```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```

* Comparator가 정렬에 대한 추상 전략이 되며, 익명 클래스가 구체적인 전략을 구현한다.
* 코드가 너무 길어지기 때문에 함수형 프로그래밍은 Java에서 적합하지 않았다.

<br>

## 2. 람다식(Lambda Expression)

Java 8 부터는 추상 메서드 하나짜리 인터페이스를 **함수형 인터페이스**로 칭하고, **람다식**을 통해 함수형 인터페이스의 인스턴스를 만들 수 있게 되었다.

> SortFourWays.java

```java
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));

Collections.sort(words, comparingInt(String::length));
```

* 컴파일러가 문맥을 추론하여 String, int 등의 필요한 타입을 추론한다.
  * 위 코드 중 words 인수가 List 로 타입이 아닌 ``List<String>``이기 때문에 추론 가능하다.
* 컴파일러가 타입을 결정하지 못할 때 개발자가 직접 명시한다.
* 타입을 명시해야 코드가 명확할 때를 제외하고는, 람다의 모든 매개변수 타입은 생략한다.

> Operation.java

```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }
}
```

* 열거 타입의 생성자에 넘겨지는 인수들의 타입 또한 컴파일타임에 추론된다.
* 반면 인스턴스는 런타임에 생성되기 때문에, 열거 타입 생성자 안의 람다는 인스턴스 필드나 메서드에 접근할 수 없다.
  * 이를 이용해야 한다면 상수별 클래스/인터페이스 몸체를 사용해야 한다.

### 2.1. 한계

람다는 메서드나 클래스와 달리 이름이 없어 문서화를 할 수 없다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드가 길어지면 사용하지 말아야 한다. 람다는 한 줄일 때 가장 좋고, 길어도 가독성을 위해 세 줄 안에는 끝내자.

* 람다는 함수형 인터페이스에서만 쓰이기 때문에 다음의 경우 익명 클래스를 사용해야 한다.
  * 추상 클래스의 인스턴스 혹은 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때.
* 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 사용하라.
  * 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
  * 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
* 람다를 직렬화하는 일은 극히 삼가하라.
  * 람다도 익명 클래스처럼 직렬화 형태가 구현별로 상이할 수 있다.
  * 직렬화해야만 하는 함수 객체(예 : Comparator)가 있다면 private 정적 중첩 클래스의 인스턴스를 사용하자.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
