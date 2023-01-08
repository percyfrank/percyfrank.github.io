---
title: "[Effective Java] Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라"
excerpt: "유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. 매개변수화 타입의 불공변

``List<String>``은 ``List<Object>``의 하위 타입이 아니다. ``List<Object>``에는 어떤 객체든 넣을 수 있지만, ``List<String>``은 오직 문자열만 취급한다. ``List<Object>``가 하는 일을 ``List<String>``으로 대체해서 할 수 없기 때문에 리스코프 치환 원칙을 위배한다.

> Stack.java

```java
public class Stack<E> {

    private E[] elements;

    //생략..

    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }

    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
        numberStack.pushAll(integers); //컴파일 에러 발생
    }
}
```

Integer는 Number의 하위 클래스이지만, 매개변수화 타입의 불공변 특징으로 인해 ``Stack<Integer>``는 ``Stack<Number>``의 하위 타입이 아니다. 따라서 ``Stack<Number>``에 Integer 원소를 담으려는 구문에서 컴파일 에러가 발생한다.

> Stack.java

```java
public void pushAll(Iterable<? extends E> src);
```

E와 E의 하위 타입을 받을 수 있음을 명시하는 한정적 와일드카드 타입을 이용하면 타입 안전해진다.

> Stack.java

```java
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

public static void main(String[] args) {
    Stack<Number> numberStack = new Stack<>();
    Collection<Object> objects = new ArrayList<>();
    numberStack.popAll(objects); //컴파일 에러 발생
}
```

``Collection<Object>``에 Number 객체를 담으려고 하기 때문에 컴파일 에러가 발생한다. ``Collection<Number>``는 ``Collection<Object>``의 하위 타입이 아니기 때문이다.

> Stack.java

```java
public void popAll(Collection<? super E> dst);
```

Collection의 매개변수 타입이 E의 상위 타입(E를 포함)이라고 명시하면 타입 안전해진다.

<br>

## 2. PECS

> 펙스(PECS) : Producer-Extends, Consumer-Super

유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라. 단, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 사용하지 않는다.

매개변수화 타입 T가 생산자라면 ``<? extends T>``를 사용하고, 소비자라면 ``<? super T>``를 사용한다. 위 Stack 예제는 E 인스턴스를 생산하고 소비하는지에 따라 PECS를 다르게 적용했다.

> Union.java

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<E>(s1);
    result.addAll(s2);
    return result;
}

public static void main(String[] args) {
    Set<Integer> integers = new HashSet<>();
    Set<Double> doubles =  new HashSet<>();
    Set<Number> numbers = union(integers, doubles);
}
```

반면 반환 타입에는 한정적 와일드카드 타입을 사용하지 않는다. 유연성을 높여주지 않고 클라이언트 코드에서도 와일드카드 타입을 사용해야 하기 때문이다.

* 만약 컴파일러가 올바른 타입을 추론하지 못할 때 명시적 타입 인수를 사용하라.

### 2.1. 재귀적 타입 한정 응용

> RecursiveTypeBound.java

```java
//기존
public static <E extends Comparable<E>> E max(List<E> list);

//PECS 적용
public static <E extends Comparable<? super E>> E max(List<? extends E> list);
```

``Comparable<E>``는 ``compareTo()``로 항상 인스턴스 E를 소비하여 선후 관계를 의미하는 정수를 생산하는 소비자이다. 따라서 ``Comparable<? super E>``로 변경한다. 입력 매개변수는 E 인스턴스를 제공하는 생산자이다.

* ``List<ScheduledFuture<?>>``는 오로지 PECS를 적용한 메서드로만 최대 값을 뽑아낼 수 있다.
  * ScheduledFuture는 Delayed 인터페이스의 하위 인터페이스고, ``Comparable<Delayed>``를 구현했기 때문이다.
* Comparator 역시 소비자이다.

<br>

## 3. 비한정적 타입 매개변수와 비한정적 와일드카드

> Swap.java

```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j); //더 낫다!
```

기본 규칙은 **메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체한다.** 따라서, public API라면 두 번째 방식이 더 낫다. 신경쓸 타입 매개변수가 없기 때문이다. 다만 ``List<?>``는 null 이외의 값을 넣을 수 없다는 문제가 있어 두 번째 방식으로 코드를 짜면 컴파일 에러가 발생한다.

> Swap.java

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

와일드카드 타입의 실제 타입을 알려주는 private 헬퍼 메서드를 사용하면 문제가 해결된다. 헬퍼 메서드는 리스트가 ``List<E>``이며 꺼내고 넣는 값들 역시 전부 E 타입이기 때문에 안전함을 알고 있다. 클라이언트는 헬퍼 메서드를 모른채 ``swap()``을 사용하게 된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
