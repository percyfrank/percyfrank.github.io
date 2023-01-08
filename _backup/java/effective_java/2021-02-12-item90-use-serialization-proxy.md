---
title: "[Effective Java] Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라"
excerpt: "제 3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-12
last_modified_at: 2021-02-12
---

## 1. 직렬화 프록시 패턴(Serialization Proxy Pattern)

숨겨진 생성자를 제공하는 직렬화로 인해 생기는 버그 및 보안 문제의 위험을 줄여주는 기법이다. 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.

* 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다.
  * 이 중첩 클래스가 바깥 클래스의 직렬화 프록시다.
  * 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다.
  * 생성자는 유효성 검사 및 방어적 복사 없이 단순히 인스턴스의 데이터를 복사한다.
  * 바깥 클래스와 직렬화 프록시 모두 Serializable을 구현한다.

> Period.SerializationProxy.java

```java
private static class SerializationProxy implements Serializable {

    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID = 234098243823485285L; // 아무 값이나 상관없다.
}
```

* 바깥 클래스와 동일한 필드로 구성되어 있다.

> Period.java

```java
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

* 바깥 클래스에 ``writeReplace()`` 메서드를 추가한다.
* 해당 메서드는 범용적이라서 직렬화 프록시를 사용하는 모든 클래스에 그대로 복사해 쓰면 된다.
* 바깥 클래스의 인스턴스 대신 프록시의 인스턴스를 반환하는 역할이다.
  * 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.
* 해당 메서드를 이용하면 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없다.

> Period.java

```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

* 공격자가 ``readObject()``로 불변식을 훼손하고자 시도할 수 있으나, 바깥 클래스에 예외를 던지는 매서드를 재정의해 공격을 막는다.

> Period.SerializationProxy.java

```java
private Object readResolve() {
    return new Period(start, end);
}
```

* 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 ``readResolve()`` 메서드를 프록시 클래스에 추가한다.
* 해당 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환한다.
* 일반 인스턴스를 만들 때와 똑같은 생성자, 정적 팩터리 등을 사용해 역직렬화된 인스턴스를 생성한다.
  * 기존 직렬화는 생성자를 이용하지 않고도 인스턴스를 생성하기 때문에 불변식을 검사해야 했지만, 프록시 패턴은 이를 검사할 필요가 없다.

<br>

## 2. 방어적 복사와의 비교

> Period.java

```java
private Date start;
private Date end;

private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    start = new Date(start.getTime());
    end = new Date(end.getTime());

    if (start.compare(end))
        throw new InvalidObjectException();  
}
```

* 방어적 복사를 사용하는 경우 Period의 필드를 final로 선언할 수 없다.
* 그러나 프록시 패턴은 필드를 final로 선언해도 되므로 진정한 불변으로 만들 수 있다.
* 프록시 패턴은 특정 필드가 직렬화 공격 대상이 되지 못하게 막으며, 역직렬화할 때 유효성 검사가 필요없다.

### 2.1. EnumSet 사례

직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.

> SerializationProxy.java

```java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {

    private final Class<E> elementType;
    private final Enum<?> [] elements;

    SerializationProxy(EnumSet<E> set) {
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]);
    }

    private Object readResolve() {
        EnumSet<E> result EnumSet.noneOf(elementType);
        for (Enum<?> e elements)
            result.add((E) e );
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}
```

* EnumSet의 팩토리는 열거 타입의 원소의 개수에 따라 두 하위 클래스 중 하나의 인스턴스를 반환한다.
* RegularEnumSet 인스턴스의 EnumSet을 직렬화한 다음, 원소를 추가하고 JumboEnumSet로 역직렬화할 수 있다.
  * EnumSet이 프록시 패턴을 사용하기에 가능하다.

<br>

## 3. 직렬화 프록시 패턴의 한계

1. 클라이언트가 멋대로 확장할 수 있는 클래스에 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다.
이런 객체의 메서드를 직렬화 프록시의 ``readResolve()`` 안에서 호출하려 하면 ClassCastException이 발생한다.
직렬화 프록시만 가졌을 뿐 실제 객체는 아직 만들어지지 않았기 때문이다.
3. 방어적 복사보다 속도가 느리다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
