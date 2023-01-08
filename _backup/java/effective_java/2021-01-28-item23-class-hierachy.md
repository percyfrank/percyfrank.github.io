---
title: "[Effective Java] Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라"
excerpt: "태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-27
last_modified_at: 2021-01-27
---

## 1. 태그 달린 클래스

> Figure.java

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    final Shape shape;

    double length;
    double width;

    double radius;

    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

Figure 클래스는 현재 객체가 원인지 사각형인지를 **열거형 태그값을** 통해 표현하고 있다. 하나의 클래스가 두 가지 이상의 의미를 담고 있기 때문에 여러 단점들이 존재한다.

* 코드가 장황하고 가독성이 나쁘다.
  * 또 다른 의미를 추가하려면 코드를 대대적으로 수정해야 한다.
* 사용하지 않는 의미의 필드가 final이라면 생성자에서 불필요하게 초기화해야 한다.
* 다른 의미를 위한 코드들이 존재함으로써 메모리를 많이 사용한다.
* 인스턴스 타입만으로는 현재 나타내는 의미를 알 수가 없다.

<br>

## 2. 클래스 계층구조

클래스 계층구조를 사용하면 태그 달린 클래스의 단점을 모두 상쇄하게 된다. 루트 클래스의 코드를 수정하지 않고도 독립적으로 계층구조를 확장해 사용할 수 있다.

> Figure.java

```java
abstract class Figure {
    abstract double area();
}
```

> Circle.java

```java
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
```

> Rectangle.java

```java
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```

* 계층구조의 루트가 되는 추상 클래스를 정의한다.
* 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 정의한다.
* 태그 값에 상관없이 동작하는 메서드들은 루트 클래스의 일반 메서드로 추가한다.
* 이후, 루트 클래스를 확장해 구체 클래스별로 의미를 정의한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
