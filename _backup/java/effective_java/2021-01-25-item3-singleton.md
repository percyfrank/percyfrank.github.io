---
title: "[Effective Java] Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라"
excerpt: "싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 싱글턴(Singleton)

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미한다. 무상태 객체나 설계상 유일해야 하는 컴포넌트가 대표적인 예이다. 싱글턴 클래스는 Mock 객체 생성이 불가능해 테스트하기 어렵다.

<br>

## 2. public static 멤버 사용

> Car.java

```java
public class Car {
    public static final Car INSTANCE = new Car();

    private Car() {
    }
}
```

Car의 private 생성자는 Car 인스턴스를 초기화할 때 한 번만 호출된다. 해당 인스턴스를 사용하기 위해서는 ``Car.INSTANCE``로 접근한다. 이를 통해 시스템상에서 해당 인스턴스는 하나뿐임이 보장된다. 이 방식은 해당 인스턴스가 싱글턴임을 API에 명시할 수 있다.

* 리플렉션 API를 사용하면 private 생성자를 호출할 수 있다.
  * 객체가 두 번 생성될 때 예외를 던짐으로써 공격을 방어할 수 있다.

<br>

## 3. 정적 팩토리 메서드 사용

> Car.java

```java
public class Car {
    private static final Car INSTANCE = new Car();

    private Car() {
    }

    public static Car getInstance() {
        return INSTANCE;
    }
}
```

* 정적 팩토리 메서드 방식은 API를 변경하지 않고도 싱글턴이 아니게 변경할 수 있다는 장점이 있다.
  * 유일한 인스턴스를 반환하던 팩토리 메서드가 스레드별로 다른 인스턴스를 제공할 수 있다.
* 정적 팩토리를 제너릭 싱글턴 팩토리로 변경할 수 있다.
* 정적 팩토리의 메소드 참조를 공급자로 사용할 수 있다.
  * ``Car::getInstance``를 ``Supplier<Car>``로 사용하는 식이다.

<br>

## 4. 직렬화

위 두 가지 방식으로 만든 싱글턴 클래스를 직렬화하기 위해서는 Serializable 구현으로 부족하다. 모든 인스턴스 필드를 ``transient``라고 선언하고 ``readResolve()`` 메서드를 제공해야 한다. 그렇지 않으면 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

<br>

## 5. 열거형

> Car.java

```java
public enum Car {
    INSTANCE;
}
```

추가 노력없이 직렬화가 가능하며 리플렉션 공격도 막는다. 단 해당 싱글턴이 Enum이 아닌 다른 클래스를 상속받아야 하는 경우 사용할 수 없다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
