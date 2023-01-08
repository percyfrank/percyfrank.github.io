---
title: "[Effective Java] Item 44. 표준 함수형 인터페이스를 사용하라"
excerpt: "입력값과 반환값에 함수형 인터페이스 타입을 활용하라."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-03
last_modified_at: 2021-02-03
---

## 1. 불필요한 함수형 인터페이스

> LinkedHashMap.java

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

* 위 메서드를 함수 객체로 리팩토링하려면 다음을 고려해야 한다.
  * 함수 객체는 Map의 인스턴스 메서드가 아니기 때문에 ``size()``라는 인스턴스 메서드를 참조할 수 없다.
  * Map 자신도 함수 객체에 넘겨주어야 한다.

> EldestEntryRemovalFunction.java

```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V> {
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

이렇게 정의할 수 있겠으나 Java는 이미 다양한 용도의 표준 함수형 인터페이스를 제공하고 있다. 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 사용한다.

* 직접 만든 함수형 인터페이스는 @FunctionalInterface 애너테이션을 달아 의도를 드러낸다.
  * 해당 클래스의 코드나 문서를 읽을 때, 해당 인터페이스가 람다용으로 설계된 것임을 알려준다.
  * 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
  * 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
* 함수형 인터페이스를 API에서 사용할 때 오버로딩을 주의하라.
  * 서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피해야 한다.
  * 클라이언트에게 모호함을 안겨주며, 코드에서 문제가 발생할 수 있다.

<br>

## 2. 표준 함수형 인터페이스

* ``UnaryOperator<T>`` (함수 시그니처 ``T apply(T t)``)
* ``BinaryOperator<T>`` (함수 시그니처 ``T apply(T t1, T t2)``)
* ``Predicate<T>`` (함수 시그니쳐 ``boolean test(T t)``)
* ``Function<T, R>`` (함수 시그니쳐 ``R apply(T t)``)
* ``Supplier<T>`` (함수 시그니쳐 ``T get()``)
* ``Consumer<T>`` (함수 시그니쳐 ``void accept(T t)``)

### 2.1. 변형

* 기본 인터페이스는 기본 타입인 int, long, double 용으로 각 3개씩 변형이 생겨난다.
  * IntPredicate, LongBinaryOperator 등.
  * Function은 LongFunction\<int[]\> 같이 반환 타입만 매개변수화한다.
* Function 인터페이스는 기본 타입을 반환하는 변형이 총 9개 있다.
  * 입력과 결과 타입이 모두 기본 타입이면 접두어에 ``SrcToResult``를 사용한다.
    * LongToIntFunction 등.
  * 입력이 객체 참조이고 결과가 기본 타입이면 접두어에 ``ToResult``를 사용한다.
    * ToLongFunction\<int[]\> 등.
* Predicate, Function, Consumer는 인수를 2개씩 받을 때 접두어에 ``Bi``를 사용한다.
  * BiPredicate\<T, U\>, BiFunction\<T, U, R\> 등.
  * BiFunction은 기본 타입을 반환할 때 똑같이 접두어에 ``ToResult``를 사용한다.
    * ToIntBiFunction\<T, U\> 등.
  * Consumer는 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형은 다음과 같다.
    * ObjIntConsumer\<T\>, ObjLongConsumer\<T\> 등.
* Supplier는 boolean을 반환할 때 BooleanSupplier를 사용할 수 있다.

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하면 성능 저하가 발생할 수 있다.

### 2.2. 전용 함수형 인터페이스 구현 조건

이 중 하나 이상을 만족한다면 표준 함수형 인터페이스가 아닌 전용 함수형 인터페이스 구현을 고려한다. 왜 ToIntBiFunction\<T,U\>를 사용하지 않고 Comparator\<T\>를 별도로 구현했을까?

* 자주 쓰이며, 이름 자체가 용도를 명확하게 설명해준다.
* 반드시 따라야 하는 규약이 있다.
* 유용한 디폴트 메서드를 제공할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
