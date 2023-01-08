---
title: "[Effective Java] Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라"
excerpt: "문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 문자열의 한계

문자열은 다른 값 타입을 대신하기에 적합하지 않다. 어느 데이터가 기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고, 없다면 새로 작성하라.

* 문자열은 열거 타입을 대신하기에 적합하지 않다.
* 문자열은 혼합 타입을 대신하기에 적합하지 않다.
  * ``String compundKey = className + "#" + i.next();``
  * 각 요소를 개별로 접근하려면 문자열을 파싱해야 한다.
  * String이 제공하는 기능에만 의존해야 한다.
* 문자열은 권한을 표현하기에 적합하지 않다.

> ThreadLocal.java

```java
public class ThreadLocal {

    private ThreadLocal() { }

    public static void set(String key, Object value);
    public static Object get(String key);
}
```

스레드별 지역 변수를 사용하기 위해 문자열 키를 사용했다. 문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다. 같은 키를 쓰게 되면 변수를 공유하게 되는 등 보안이 취약해진다.

> ThreadLocal.java

```java
public class ThreadLocal {

    private ThreadLocal() { }

    public static class Key {
        Key() { }
    }

    public static Key getKey() {
        return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

* 문자열 대신 위조할 수 없는 키를 사용하면 해결된다.

> ThreadLocal.java

```java
public final class ThreadLocal<T> {

    public ThreadLocal() { }
    public void set(T value);
    public T get();
}
```

* setter와 getter는 Key 클래스의 인스턴스 메서드로 바꾼다.
  * Key는 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.
* ThreadLocal은 하는 일이 없어지므로 치워버리고, 중첩 클래스 Key 이름을 ThreadLocal로 변경한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
