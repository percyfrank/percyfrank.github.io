---
title: "[Effective Java] Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라"
excerpt: "제네릭과 가변인수는 궁합이 좋지 않다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. 가변인수

가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 생성된다. varargs 매개변수에 제네릭이나 타입 매개변수가 포함되면 실체화 불가 타입의 배열을 사용하기 때문에 컴파일 경고가 발생한다.

### 1.1. 힙 오염

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 다른 타입의 객체를 참조하는 상황에서 컴파일러는 자동 생성한 형변환이 실패할 수 있고, 타입 안전하지 못해진다.

> Dangerous.java

```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException 발생
}
```

* 위 메서드는 형변환을 하지 않음에도 런타임 에러가 발생한다.
* 마지막 줄에 컴파일러가 보이지 않는 형변환을 삽입하기 때문이다.
* 타입 안정성이 깨져 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 못하다.

<br>

## 2. @SafeVarargs

Java 7 이전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해 해줄 수 있는 일이 없었다. 호출자는 호출하는 곳마다 @SuppressedWarnings("unchecked") 애너테이션을 일일이 달아서 경고를 숨겨야 했다.

Java 7 부터는 메서드 작성자가 @SafeVarargs 애너테이션을 달아 해당 메서드가 타입 안전함을 보장하도록 하여 컴파일러의 경고를 숨길 수 있다. 안전함이 확실히 보장되지 않으면 해당 애너테이션을 사용하면 안 된다.

* 메서드가 안전한지 확신하는 방법이란?
  * 메서드는 varargs 매개변수를 담는 제네릭 배열에 아무것도 저장하지 않는다.
    * 매개변수들을 덮어 쓰지 않는다.
  * 해당 배열의 참조가 외부에 노출되지 않는다.
    * 신뢰할 수 없는 코드가 배열에 접근할 수 없다.
* varargs 매개변수 배열이 호출자로부터 메서드로 인수들을 전달하는 일만 하면 안전하다.
* @SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다.

> PickTwo.java

```java
public class PickTwo {

    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한"); // ClassCastException 발생
    }
}
```

* ``toArray()`` 메서드는 매개변수 배열을 그대로 반환하여 힙 오염이 호출자의 콜 스택으로 전이될 수 있다.
* ``pickTwo()`` 메서드는 ``toArray()`` 메서드를 통해 항상 Object[]를 반환한다.
* 그러나 호출자 쪽에서 형변환이 실패하여 예외가 발생한다.
  * 컴파일러가 자동으로 String[] 형변환 코드를 삽입한다.
  * Object[]는 String[]의 하위 타입이 아니기 때문에 실패한다.
* varargs 매개변수 배열의 참조가 외부에 노출되어 안전하지 않다.

### 2.1. 예외 사항

**일반적으로 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 타입 안전하지 않다.** 하지만 예외 사항이 존재한다.

* @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드로 넘기는 것은 안전한다.
* 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

> Flatten.java

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
```

* 애너테이션이 달려 있어 선언하는 쪽과 사용하는 쪽 모두 안전하게 사용이 가능하다.

> Flatten.java

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}

public static void main(String[] args) {
    audience = flatten(List.of("a", "b", "c"));
}
```

* 실체는 배열인 varargs 매개변수를 List 매개변수로 변환해서 사용하면 안전하다.
* ``List.of()``에도 @SafeVarargs 애너테이션이 달려있다.

> SafePickTwo.java

```java
public class SafePickTwo {

    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
    }
}
```

* 기존의 ``pickTwo()``는 ``toArray()``를 통해 varargs 매개변수 배열에 직접 접근하고 이를 외부에 노출시켰다.
* ``List.of()``를 통해 배열 없이 제네릭만 사용하기 때문에 안전하다.

<br>

## 3. 요약

가변인수 기능은 배열을 노출하여 추상화가 완벽하지 않고, 배열과 제네릭의 규칙이 다르기 때문에 궁합이 좋지 않다. 제네릭 varargs 매개변수를 사용하려면 메서드의 타입 안전을 확인하고 @SafeVarargs 애너테이션을 붙이자.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
