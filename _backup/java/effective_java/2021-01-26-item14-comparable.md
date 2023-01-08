---
title: "[Effective Java] Item 14. Comparable을 구현할지 고려하라"
excerpt: "Comparable을 구현하면 컬렉션의 정렬 및 검색 등 유용한 기능을 사용할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-26
last_modified_at: 2021-01-26
---

## 1. Comparable

> ``compareTo()`` 메서드는 Object에 정의되어 있지 않고, 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 제네릭하다.

인스턴스들이 자연적인 순서(natural order)가 있다면 Comparable을 구현하자. 해당 인터페이스를 활용하는 많은 제네릭 알고리즘 및 컬렉션들이 제공하는 정렬 및 검색 등의 유용한 기능을 사용할 수 있다. 많은 Java 플랫폼 라이브러리의 값 클래스 및 열거 타입이 Comparable을 구현했다.

<br>

## 2. compareTo 메서드의 일반 규약

> Comparable.java

```java
public interface Comparable<T> {

    public int compareTo(T o);
}
```

객체가 주어진 객체보다 작은 경우 음수를, 큰 경우 양수를, 같으면 0을 반환한다. equals와 마찬가지로 반사성, 대칭성, 추이성 규약을 지켜야 한다.

* 두 객체의 참조 순서를 바꿔 비교하더라도 예상한 결과가 나와야 한다.
* 첫 번째 객체가 두 번째 객체보다 크고 두 번째 객체가 세 번째 객체보다 크다면, 첫 번째 객체는 세 번째 객체보다 커야한다.
* 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.
  * ``compareTo()``로 비교한 동치성 결과가 ``equals()``와 같아야 한다.
  * 일부 정렬된 컬렉션은 동치성을 비교할 때 ``equals()`` 대신 ``compareTo()``를 사용하기 때문이다.

제너릭 인터페이스이기 때문에 ``compareTo()`` 인수 타입은 컴파일 시점에 정해진다. 인수의 타입이 잘못되었다면 예외가 발생하기 때문에 별도의 타입 체크 혹은 형변환이 필요없다.

여러 필드들을 비교할 때는 성능을 위해 중요도가 높은(차이가 날 확률이 높은) 핵심 필드부터 비교하도록 코드를 작성하자.

<br>

## 3. Comparator

Comparable을 구현하지 않은 필드 혹은 표준이 아닌 순서로 비교해야 한다면 **Comparator**를 사용하자. 람다식을 통해 코드가 간결해지지만 약간의 성능 저하가 유발된다.

> PhoneNumber.java

```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.prefix)
                .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

그러나 아래와 같이 값의 차를 기준으로 순서를 정하는 방식은 지양하자.

> HashComparator.java

```java
static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    @Override
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

정수 오버플로우나 부동 소수점 계산으로 인한 오류가 발생할 수 있다.

> HashComparator.java

```java
private static Comparator<Object> hashCodeOrder = Comparator.comparingInt(Object::hashCode);

//혹은

private static Comparator<Object> hashCodeOrder = new Comparator<Object>() {
    @Override
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

대신 박싱된 기본 타입 클래스가 제공하는 ``compare()``나 Comparator가 제공하는 비교자 생성 메서드를 사용한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
