---
title: "[Effective Java] Item 7. 다 쓴 객체 참조를 해제하라"
excerpt: "메모리 누수는 항상 신경써야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 다 쓴 참조(Obsolete Reference)

> Stack.java

```java
public class Stack {

    private final Object[] arr = new Object[30];
    private int size = 0;

    public void add(Object element) {
        checkSize();
        arr[size++] = element;
    }

    public Object pop() {
        return arr[--size];
    }

    private void checkSize() {
        if (arr.length == size) {
            arr = Arrays.copyOf(arr, size * 2 + 1);
        }
    }
}
```

스택이 꽉 차면 ``Arrays.copyOf()``를 호출하여 더 큰 사이즈의 스택을 만들고 기존 스택의 값들을 복사한다. ``pop()``을 통해 스택에서 객체를 꺼내고 프로그램이 꺼내진 객체들을 더이상 사용하지 않아도 GC는 이들을 회수하지 않는다. 스택이 해당 꺼내진 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다.

* 다 쓴 참조란 앞으로 다시 쓰지 않을 참조를 의미한다.
  * 위 코드의 배열에서 활성 영역(인덱스가 size보다 작은 원소들) 밖의 참조들이 해당된다.

객체 참조가 하나라도 살아있다면 GC는 해당 객체와 해당 객체가 참조하는 모든 객체들을 회수할 수 없다. 이는 성능의 문제로 이어진다.

<br>

## 2. 해결 방법

> Stack.java

```java
public Object pop() {
    Object object = arr[--size];
    arr[size] = null;
    return object;
}
```

해당 참조를 다 썼을 때 null을 대입하여 참조를 해제한다. 위 클래스에서 원소의 참조가 필요없어지는 시점은 스택에서 꺼냈을 때이다.

그러나 객체 참조를 null로 처리하는 일은 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 좋은 방법은 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.

<br>

## 3. 예시 Stack 클래스의 문제점

위 예시 Stack 클래스가 메모리 누수에 취약한 이유는?

* arr 배열로 저장소 풀을 만들어 원소들을 관리하는 등 Stack 클래스가 자기 메모리를 관리하기 때문이다.
* 개발자는 비활성 영역의 원소들이 더이상 쓸모 없다는 것을 안다.
* 그러나 GC에게 비활성 영역의 원소들도 유효한 객체로 인식된다.

자기 메모리를 직접 관리하는 클래스는 개발자가 메모리 누수에 신경써야 한다. 원소를 다 사용한 즉시 null을 처리하여 GC에게 더이상 참조하지 않음을 명시한다.

<br>

## 4. 캐시 및 콜백 리스너

캐시는 메모리 누수의 주범이다. 일반적으로 캐시를 만들 때 캐시 엔트리의 유효 기간을 정확히 정의하기 어렵다. 콜백 리스너 또한 클라이언트가 등록하고 명확하게 해지하지 않으면 계속 쌓여간다.

* 캐시 외부에서 Key를 참조하는 동안만 엔트리가 살아있는 캐시가 필요하다면 WeakHashMap을 사용한다.
* 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 사용한다.
  * 스레드나 부수 작업으로 오래된 엔트리를 청소한다.
* 콜백 리스너를 약한 참조(WeakHashMap 등록)로 저장하면 GC가 수거해간다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
