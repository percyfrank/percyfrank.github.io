---
title: "[Effective Java] Item 30. 이왕이면 제네릭 메서드로 만들라"
excerpt: "메서드 또한 형변환 없이 사용할 수 있는 제네릭 메서드가 편하다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. 제네릭 메서드

> Union.java

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

* 제네릭 싱글턴 팩토리 : 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩토리이다.
  * ``Collections.reverseOrder()`` 같은 함수 객체들이 사용한다.

### 1.1. 항등 함수

> GenericSingletonFactory.java

```java
public class GenericSingletonFactory {
    private static final UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
}
```

강제 형변환할 때 컴파일러가 비검사 예외를 경고한다. ``UnaryOperator<Object>``는 ``UnaryOperator<T>``가 아니기 때문이다. 그러나 항등 함수란 상태가 없으며 입력 값을 수정 없이 그대로 반환하는 메서드이다. T가 어떤 타입이든 ``UnaryOperator<Object>``를 사용해도 타입 안전하기 때문에 애너테이션을 붙여준다.

### 1.2. 재귀적 타입 한정

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.

> RecursiveTypeBound.java

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
      //...
}
```

Comparable을 구현한 원소의 컬렉션을 받는 메서드는 해당 원소들에 대해 정렬, 검색, 최대, 최소 등의 작업을 수행한다. 이를 위해 컬렉션에 담긴 원소들은 모두 상호 비교가 가능해야 한다. ``<E extends Comparable<E>>``는 모든 타입 E는 자신과 비교할 수 있다는 의미를 잘 표시하고 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
