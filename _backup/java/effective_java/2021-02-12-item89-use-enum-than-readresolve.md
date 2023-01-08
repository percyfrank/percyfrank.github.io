---
title: "[Effective Java] Item 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라"
excerpt: "불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-12
last_modified_at: 2021-02-12
---

## 1. 싱글턴과 직렬화

> Car.java

```java
public class Car {
    public static final Car INSTANCE = new Car();

    private Car() {
    }
}
```

외부에서 생성자를 호출하지 못하게 하더라도 직렬화를 구현하는 순간 싱글턴이 아니게 된다. 어떤 ``readObject()``를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환한다.

> Car.java

```java
private Object readResolve() {
    return INSTANCE;  
}
```

``readResolve()``를 사용하면 ``readObject()``가 만들어낸 인스턴스를 다른 것으로 대체할 수 있다. 역직렬화된 새 인스턴스는 참조를 잃어 자동적으로 GC 대상이 된다.

* 해당 싱글턴 인스턴스의 직렬화 형태는 실 데이터를 가질 이유가 없으므로, 모든 인스턴스 필드를 transient로 선언해야 한다.
  * 즉, ``readResolve()``를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드를 모두 transient로 선언하자.
* 그렇지 않으면 역직렬화된 객체의 참조를 공격할 여지가 남는다.
  * 싱글턴이 transient가 아닌 참조 필드를 가지고 있다면, 필드의 내용은 ``readResolve()`` 실행 전에 역직렬화된다.
  * 잘 조작된 스트림을 사용하면 해당 필드의 내용이 역직렬화되는 시점에 해당 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

<br>

## 2. 열거형과 직렬화

``readResolve()`` 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 쓰기 어렵다. 싱글턴의 모든 인스턴스 필드에 transient를 붙이면 해결되지만, 열거 타입을 사용하는 것이 더 낫다.

> Elvis.java

```java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

* 직렬화 가능한 인스턴스 통제 클래스를 열거 타입으로 구현한다면 선언 상수 외의 다른 객체는 존재하지 않음을 Java가 보장한다.
  * 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 방어가 무력화된다.

<br>

## 3. 주의사항

직렬화 가능 인스턴스 통제 클래스를 작성할 때, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 경우 열거 타입으로 표현하기 불가능하기 때문에 ``readResolve()``를 써야한다.

``readResolve()``의 접근성은 매우 중요하다. final 클래스라면 해당 메서드는 private이어야 한다.

* final이 아닌 클래스는 하기 사항들을 고려해야 한다.
  * private으로 선언하면 하위 클래스에서 사용할 수 없다.
  * protected나 public으로 선언하면 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
  * protected나 public이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
