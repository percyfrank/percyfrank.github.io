---
title: "[Effective Java] Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라"
excerpt: "매개변수가 많으면 각각이 의미하는 바가 무엇인지 이해하기 어렵다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. 점층적 생성자 패턴

> Car.java


```java
public class Car {

    private final int a;
    private final int b;
    //...
    private final int e;

    public Car(int a, int b, int c, int d, int e) {
        this.a = a;
        this.b = b;
        this.c = c;
        this.d = d;
        this.e = e;
    }

    public Car(int a, int b, int c, int d) {
        this(a, b, c, d, 0);
    }

    public Car(int a, int b, int c) {
        this(a, b, c, 0);
    }

    public Car(int a, int b) {
        this(a, b, 0);
    }
}
```

필수 매개 변수만 받는 생성자와 필수 매개 변수 및 선택 매개 변수를 받는 생성자들을 점층적으로 만들어서 사용하는 패턴이다. 원하는 매개변수를 포함한 생성자 중 가장 짧은 것을 사용하여 인스턴스를 제작한다.

매개변수가 많아지면 클라이언트가 코드를 읽고 쓰는 것이 어려워진다. 각 값이 어떤 의미인지 한 번에 파악하기 어렵기 때문이다.

<br>

## 2. Java Beans 패턴

> Car.java

```java
public class Car {

    private int a = 0;
    private int b = 0;
    private int c = 0;
    private int d = 0;
    private int e = 0;

    public Car() {
    }

    //각 필드들에 대한 setter
}
```

각 필드들에 기본값을 주고 객체를 생성할 때 setter를 사용한다.

> Main.java

```java
Car car = new Car();
car.setA(1);
car.setB(100);
//...
car.setE(-1);
```

객체를 생성하기 위해 여러 메서드를 호출해야 하며, 객체가 완전히 생성되기 전까지는 일관성이 파괴되는 단점이 있다. 또한 클래스를 불변으로 생성할 수 없기 때문에 스레드 안정성 추가 작업이 필요하다.

<br>

## 3. Builder 패턴

> Car.java

```java
public class Car {

    private final int a;
    private final int b;
    private final int c;
    private final int d;

    public Car(Builder builder) {
        a = builder.a;
        b = builder.b;
        c = builder.c;
        d = builder.d;
    }

    public static class Builder {

        //필수 매개변수
        private final int a;
        private final int b;

        //선택 매개변수
        private int c = 0;
        private int d = 0;

        public Builder(int a, int b) {
            this.a = a;
            this.b = b;
        }

        public Builder c(int c) {
            this.c = c;
            return this;
        }

        public Builder d(int d) {
            this.d = d;
            return this;
        }

        public Car build() {
            return new Car(this);
        }
    }
}
```

Car 클래스는 불변이며, 내부 Builder 클래스의 메서드 체이닝을 응용한다.

> Main.java

```java
Car car = new Car.Builder(3, 0)
        .c(-1)
        .d(1000)
        .build();

Car car2 = new Car.Builder(1, -1)
        .d(99)
        .build();
```

필수 매개변수는 Builder 객체를 생성할 때 대입하며, 선택 매개 변수는 내부 Builder 클래스에 정의된 매칭 메서드를 이용한다. 이를 통해 현재 사용자가 대입하는 값이 어떤 매개변수에 대입되는지 간단하게 확인할 수 있다.

빌더 패턴은 계층적으로 설계된 클래스에서 사용하기 좋다. 생성 비용이 큰 것은 아니지만 성능에 민감한 상황에서는 문제가 될 수 있다. 또한 매개변수가 4개 이상은 되어야 효율적이다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
