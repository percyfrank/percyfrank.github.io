---
title: "[Effective Java] Item 55. 옵셔널 반환은 신중히 하라"
excerpt: "성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. Optional

메서드가 특정 조건에서 값을 반환할 수 없을 때 기존에는 예외를 던지거나 null을 반환했다. 그러나 이 두 방법은 다음과 같은 한계를 가진다.

* 예외는 진짜 예외적인 상황에서만 사용해야 한다.
  * 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 크다.
* null을 반환하면 해당 메서드를 사용하는 코드들은 null 방어 코드를 추가해야 한다.

> Max.java

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);
    return Optional.of(result);
}
```

* Java 8 부터는 Optional\<T\>를 통해 null이 아닌 T 타입 참조를 하나 담아 반환한다.
  * 반환할 값이 없을 때는 ``Optional.empty()``를 통해 빈 옵셔널을 반환한다.
* 예외를 던지는 메서드보다 유연하고 사용하기 쉽고, null을 반환하는 메서드보다 오류 가능성이 작다.
* ``Optional.of()``에 null을 담으면 예외가 발생한다.
  * null을 허용하려거든 ``Optional.ofNullable()``을 사용한다.
* 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 않는다.

<br>

## 2. 선택 기준

null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택하는 기준이란? 옵셔널은 검사 예외와 취지가 비슷하다.

* 반환값이 없을 수도 있음을 API 사용자에게 명시한다.
  * 비검사 예외를 던지거나 null을 반환하는데 API 사용자가 인지하지 못하면 끔찍한 결과로 이어진다.
* 검사 예외를 통해 클라이언트는 반드시 이에 대처하는 코드를 작성하게 된다.

> Max.java

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

* 많은 스트림 종단 연산들은 옵셔널을 반환한다.

### 2.1. 옵셔널 활용 : 기본값 설정

> Main.java

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

* 클라이언트가 값을 받지 못했을 때 기본값을 설정할 수 있다.
* ``orElse()`` 메서드는 해당 옵셔널 값이 null이든 말든 항상 실행된다.

### 2.2. 옵셔널 활용 : 예외 호출

> Main.java

```java
String lastWordInLexicon = max(words).orElseThrow(IllegalArgumentException::new);
```

* 상황에 맞는 예외를 던진다.
* 예외 팩토리를 제공했기 때문에 실제로 예외가 발생하지 않는한 예외 생성 비용은 들지 않는다.

### 2.3. 옵셔널 활용 : 확실한 경우

> Main.java

```java
String lastWordInLexicon = max(words).get();
```

* 옵셔널에 항상 값이 채워져 있다고 확신한다면 곧바로 값을 꺼내 사용할 수 있다.

### 2.4. 옵셔널 활용 : 기본값 설정 비용이 큰 경우

> Main.java

```java
String lastWordInLexicon = max(words).orElseGet(() -> Max.findLexicon(42));
```

* 기본값 설정 비용이 아주 큰 경우 Supplier를 인수로 받는 ``orElseGet()``을 사용한다.
* ``orElseGet()``은 해당 옵셔널 값이 null일 때만 호출된다.
  * 값이 처음 필요할 때 Supplier를 사용해 생성하므로 초기 설정 비용을 낮춘다.

<br>

## 3. 옵셔널 스트림

옵셔널들의 스트림 중 채워진 옵셔널들에서 값을 뽑아 타입을 변환하는 경우가 많다.

> StreamOptional.java

```java
streamOfOptional.filter(Optional::isPresent)
                .map(Optional::get)

//Java 9 부터는 아래 코드를 사용할 수 있다.
streamOfOptional.flatMap(Optional::stream);
```

Java 9 부터는 Optional에 ``stream()`` 메서드가 추가되었다. Optional을 Stream으로 변환해주는 어댑터이다. 옵셔널에 값이 있으면 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로 변환한다.

<br>

## 4. 주의사항

컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다.

* Optional\<List\<T\>\>를 반환하기 보다는 빈 List\<T\>를 반환하는 게 좋다.
  * 빈 컨테이너를 그대로 반환하면 클라이언트는 옵셔널 처리 코드가 필요없어진다.

결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 할 때 옵셔널을 반환한다.

* 그러나 옵셔널 객체를 초기화하고 값을 꺼낼 때 메서드를 호출하는 등 비용이 발생한다.
  * 성능이 중요한 경우 맞지 않을 수 있다.
* 박싱된 기본 타입을 답는 옵셔널은 래핑을 2번하기 때문에 기본 타입보다 무겁다.
  * OptionalInt, OptionalLong 등 기본 타입 전용 옵셔널 클래스를 사용한다.
  * 박싱된 기본 타입을 담은 옵셔널을 반환하지 않아야 한다.
    * 단, Boolean, Byte, Short 등 덜 중요한 기본 타입 용은 예외일 수 있다.

옵셔널을 맵의 값으로 사용하면 절대 안 된다.

* 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 된다.
* 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적절하지 않은 경우가 대다수다.

옵셔널을 인스턴스 필드에 저장해두는 것은 일반적으로 좋지는 않다.

* 필수 필드를 갖는 클래스와 이를 확장해 선택적 필드를 추가한 하위 클래스를 따로 만들어야 함을 암시하기 때문이다.
  * 그러나 기본 타입 필드들을 getter로 접근할 때 값이 없음을 나타내기 쉬워진다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
