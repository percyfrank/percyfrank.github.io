---
title: "[Effective Java] Item 53. 가변인수는 신중히 사용하라"
excerpt: "가변인수를 사용할 때는 성능 문제까지 고려하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. 가변인수(Varargs)

가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

> Varargs.java

```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

* 위 코드의 한계점은?
  * 인수를 0개만 넣어 호출하면 컴파일타임이 아닌 런타임에서 실패한다.
  * 명시적으로 유효성 검사를 진행해야 하며, for-each를 사용하지 못한다.

> Varargs.java

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```

* 필수 매개변수는 가변인수 앞에 두면 코드를 깔끔하게 리팩토링할 수 있다.
* printf와 리플렉션은 모두 가변인수를 응용하는 아이템들이다.

<br>

## 2. 주의점

가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다. 성능에 민감한 상황이라면 조심해야 한다.

> Foo.java

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

* 메서드 호출 중 95%가 인수를 3개 이하로 사용하면 해당 부분을 다중정의 한다.
* 메서드 호출 중 5%만이 배열(가변인수)를 사용한다.
  * EnumSet도 이러한 패턴을 이용해 열거 타입 집합 생성 비용을 최소화한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
