---
title: "[Effective Java] Item 1. 생성자 대신 정적 팩토리 메서드를 고려하라"
excerpt: "이름을 가질 수 있는 생성자의 장점이란?"
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 정적 팩토리 메서드(Static Factory Method)

정적 팩토리 메서드란 클래스의 인스턴스를 반환하는 정적 메서드다. 정적 팩토리 메서드의 장점은 다음과 같다.

* 이름을 가질 수 있다.
* 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
* 반환 타입의 하위 타입의 객체를 반환할 수 있는 능력이 있다.
  * 입력 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
* 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

자주 사용하는 정적 팩토리 메서드의 명명 규칙은 다음과 같다.

* from, of, valueOf
* getInstance, newInstance, create
* getType, newType, type
  * "Type"은 정적 팩토리 메서드가 반환하는 객체의 타입이다.

> Car.java

```java
public Car(int fuelEfficiency, int maximumSpeed, int weight) {
    this.fuelEfficiency = fuelEfficiency;
    this.maximumSpeed = maximumSpeed;
    this.weight = weight;
}

public static Car getSmallCar() {
    return new Car(50, 130, 800);
}

public static Car getBigCar() {
    return new Car(40, 150, 1000);
}
```

Car 클래스의 기본 생성자와 정적 팩토리 메서드가 위와 같이 정의되어 있다고 가정해보자.

<br>

## 2. 장점 1 : 이름을 가질 수 있다

> Main.java

```java
Car smallCar = new Car(50, 130, 800);
Car bigCar = new Car(30, 170, 1300);
```

자동차의 크기에 따라서 연비와 최대 속력 및 무게를 다르게 지정하여 Car 객체를 생성하는 코드이다. 매개 변수와 생성자 자체만으로는 반환될 Car 객체의 특성을 제대로 설명하지 못한다.

> Main.java

```java
Car smallCar = Car.getSmallCar();
Car bigCar = Car.getBigCar();
```

그러나 정적 팩토리 메서드는 이름을 가짐으로써 반환될 객체의 특성을 잘 설명할 수 있는 등 가독성의 장점이 있다.

<br>

## 3. 장점 2 : 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다

new 연산을 호출하면 매번 객체를 새로 생성하게 된다. 생성 비용이 큰 객체나 불변 클래스는 인스턴스를 미리 캐싱해두고 재활용할 수 있다.

> Car.java

```java
public class Car {
    private static final Car SMALL_CAR = new Car(50, 130, 800);
    private static final Car BIG_CAR = new Car(40, 150, 1000);

    //중략

    public static Car valueOf(boolean isBig) {
        return isBig ? BIG_CAR : SMALL_CAR;
    }
}
```

불필요한 객체 생성을 피할 수 있으며, 언제 어느 인스턴스를 살아 있게 할지 철저하게 통제할 수 있다.

* 클래스를 싱글턴으로 만들 수 있다.
  * 인스턴스의 생성자를 private으로 두어 인스턴스화 불가능하게 설정할 수 있다.
* 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
  * ``a == b``일 때만 ``a.equals(b)``가 성립한다.

<br>

## 4. 장점 3 : 반환 타입의 하위 타입의 객체를 반환할 수 있는 능력이 있다

> Car.java

```java
public static Car valueOf(boolean isBig) {
    return isBig ? new BigCar() : new SmallCar();
}
```

구현 클래스를 공개하지 않고도 해당 객체를 반환할 수 있다. Java에서 Collections 클래스의 정적 팩토리 메서드를 통해 45개의 유틸리티 구현체를 획득할 수 있다. 다양한 객체를 하나의 인터페이스로 다룰 수 있는 유연성 장점이 있다

<br>

## 5. 장점 4 : 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

JDBC와 같은 서비스 제공자 프레임워크를 만드는 근간이 된다.

* 서비스 인터페이스 : 구현체의 동작을 정의한다.
* 제공자 등록 API : 제공자가 구현체를 등록할 때 사용하는 API이다.
* 서비스 접근 API : 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 API이다.

클라이언트가 서비스 접근 API를 통해 원하는 구현체의 조건을 명시하면 그에 맞는 구현체를 반환한다. 구현체 제공을 프레임워크가 통제함으로써 구현체로부터 클라이언트를 분리한다.

<br>

## 6. 단점

정적 팩토리 메서드만 제공하면 하위 클래스를 생성할 수 없다. 상속을 위해서는 public 혹은 protected 생성자가 필요하다. 또한 생성자처럼 API 설명에 명확하게 드러나지 않기 때문에 개발자가 찾기 어렵다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
