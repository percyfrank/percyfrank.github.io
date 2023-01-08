---
title: "[Effective Java] Item 83. 지연 초기화는 신중히 사용하라"
excerpt: "대부분의 상황에서는 final로 필드를 선언하는 일반적인 초기화가 지연 초기화보다 낫다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 지연 초기화

지연 초기화란 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 유용하다. 그러나 대부분의 상황에서는 final로 필드를 선언하는 일반적인 초기화가 지연 초기화보다 낫다.

성능을 위해 혹은 위험한 초기화 순한을 막고자 한다면 지연 초기화를 고려한다. 단, 멀티스레드 환경에서는 지연 초기화하는 필드를 동기화해야 하기 때문에 지연 초기화를 적용하기 까다롭다.

> Initialization.java

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
```

* 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용한다.

<br>

## 2. 홀더 클래스

> Initialization.java

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field; }
```

* 성능 때문에 정적 필드를 지연초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용한다.
* ``getField()``가 처음 호출되는 순간 비로소 FieldHolder 클래스 초기화가 촉발된다.
* 일반적으로 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화한다.
* 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.
  * 동기화를 하지 않으니 성능이 느려질 거리가 없다.

<br>

## 3. 이중 검사

> Initialization.java

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null)    // 첫 번째 검사 (락 사용 안 함)
        return result;

    synchronized(this) {
        if (field == null) // 두 번째 검사 (락 사용)
            field = computeFieldValue();
        return field;
    }
}
```

* 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중 검사 관용구를 사용한다.
* 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
* 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 volatile로 선언한다.

### 3.1. 단일 검사

> Initialization.java

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null)
        field = result = computeFieldValue();
    return result;
}
```

* 반복해서 초기화해도 상관없는 인스턴스 필드는 이중 검사에서 두 번째 검사를 생략할 수 있다.
* 모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일 검사의 필드 선언에서 volatile을 삭제해도 된다.
  * 어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
