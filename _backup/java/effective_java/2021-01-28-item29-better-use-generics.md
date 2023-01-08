---
title: "[Effective Java] Item 29. 이왕이면 제네릭 타입으로 만들라"
excerpt: "클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 안전하고 편하다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. Object[] 기반의 스택 리팩토링

> Stack.java

```java
public class Stack {
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    private Object[] elements;
    private int size = 0;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

기존 스택 클래스는 사용하는 곳에서 항상 형 변환을 해야하는 번거로움이 있다. 제네릭스를 사용하여 리팩토링한다.

### 1.1. 첫 번째 방법

> Stack.java

```java
public class Stack<E> {
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    private E[] elements;
    private int size = 0;

    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

E와 같은 실체화 불가 타입으로는 배열을 만들 수 없으나, Object 배열을 강제로 ``E[]``로 형 변환하면 컴파일 할 수 있다. 다만 컴파일러가 런타임시 타입 안정성을 보장할 수 없기 때문에 비검사 예외가 발생한다.

* 해당 배열의 런타임 타입은 공변 및 실체화 특성상 ``Object[]``이다.

그러나 해당 클래스는 사용하는 원소의 타입이 항상 E임이 보장되기 때문에, 런타임시 ClassCastException이 발생 할 일이 없다. 따라서 애노테이션으로 경고 메시지를 무시하게 만든다.

### 1.2. 두 번째 방법

> Stack.java

```java
public class Stack<E> {
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    private Object[] elements;
    private int size = 0;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        @SuppressWarnings("unchecked") E result = (E) elements[--size];
        elements[size] = null;
        return result;
    }
}
```

다음 방법은 배열 자체는 ``Object[]``로 두고 원소를 뽑을 때만 제네릭 E로 비검사 형변환한다. ``push()``에서 E 타입만 허용하기 때문에 ``pop()``에서의 형변환 역시 안전하다. 애노테이션을 생성자에 두는 것이 아니라 형변환을 수행하는 할당문 내부로 옮겼다.

### 1.3. 어떤 방식이 나은가?

* 첫 번째 방법은 형변환을 생성자에서 배열 생성시 한 번만 한다.
  * 배열의 타입 ``E[]``는 E 타입 인스턴스를 받는다는 점을 직관적으로 표현한다.
  * 코드가 짧아 가독성이 좋다.
  * 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염이 발생한다.
* 두 번째 방법은 배열에서 원소를 읽을 때마다 형변환을 해야 한다.

아직은 제네릭 타입 매개변수에 기본 자료형은 사용이 불가능하여 박싱된 래퍼 클래스를 사용한다.

<br>

## 2. 요약

클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 안전하고 편하다. 기존 클라이언트에 아무 영향을 주지 않으면서 새로운 사용자를 편하게 해주도록 설계하라.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
