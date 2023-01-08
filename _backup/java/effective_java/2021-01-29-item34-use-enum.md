---
title: "[Effective Java] Item 34. int 상수 대신 열거 타입을 사용하라"
excerpt: "열거 타입은 상수를 특정 데이터와 연결지으며 상수마다 다르게 동작하도록 강제할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. 열거 타입

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다. 일반적으로 int 상수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.

### 1.1. 정수 열거 패턴의 단점

> Sample.java

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_SMALL = 1;
public static final int ORANGE_BIG = 2;
public static final int ORANGE_PIPPING = 3;
```

정수 열거 패턴은 단순히 평범한 상수를 클래스에 변수로 나열한 것이다.

* APPLE 상수가 필요한 자리에 ORANGE 상수가 대입되어도 컴파일러는 확인할 수 없다.
* 컴파일하면 상수의 값이 클라이언트 파일에 그대로 새겨진다.
  * 상수 값이 변경되면 클라이언트도 다시 컴파일해야 한다.
* 문자열로 출력하기 힘들다.
  * 의미 없는 숫자만 나열되기 때문이다.
* 문자열 상수 열거 패턴은 더 나쁜 패턴이다.
  * 하드코딩한 값에 의존하게 되며, 오타가 있더라도 컴파일러가 확인할 수 없어 런타임 에러에 노출된다.
  * 문자열 비교로 인해 성능이 저하된다.

### 1.2. 열거 타입

> Fruit.java

```java
public enum Apple { FUJI, SMALL, BIG }
public enum Orange { PIPPING, KING, WHAP_HE }
```

Java의 열거 타입 자체는 추상 클래스이며 상수 하나당 인스턴스를 만들어 public static final로 공개한다.

* 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않아 클라이언트가 확장할 수 없다.
  * 즉, 열거 타입의 인스턴스들은 싱글턴임이 보장된다.
* 열거 타입은 컴파일타임 타입 안정성을 제공한다.
  * Apple 타입 매개변수를 받는 메서드에 Orange 인스턴스를 대입하면 컴파일 에러가 난다.
* 열거 타입은 각자의 이름 공간이 있다.
  * 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일할 필요가 없다.
    * 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다.
* ``toString()``을 통해 출력하기 적합한 문자열을 내어준다.
* 메서드나 필드를 추가할 수 있다.
  * Object, Comparable, Serializable을 잘 구현해두었다.

**열거 타입의 가장 큰 장점은 상수를 특정 데이터와 연결지으며 상수마다 다르게 동작하도록 강제할 수 있다.**

> Planet.java

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```

열거 타입은 기본적으로 불변이라 모든 필드가 final이다. 필드 또한 일반 클래스처럼 private 캡슐화한다. 생성자에 데이터를 받아 인스턴스 필드에 저장하면 그만이다.

* 상수를 제거하면 제거된 상수를 참조하는 클라이언트는 다시 컴파일할 때 컴파일 오류가 발생한다.
  * 다시 컴파일하지 않더라도 런타임 시점에서 예외가 발생한다.
  * 정수 열거 패턴에서는 기대할 수 없는 효과다.
* 열거 타입을 선언한 클래스 혹은 패키지에서만 유용한 기능은 private 혹은 package-private을 선언한다.
  * 일반 클래스와 마찬가지로 클라이언트에 API를 노출할 필요가 없으면 노출을 지양한다.
* 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 클래스에서만 사용되면 멤버 클래스로 만든다.

<br>

## 2. 상수별 메서드 구현

> Operator.java

```java
public enum Operator implements OperatorService {
    PlUS("+") {
        @Override
        public int operate(int firstNumber, int nextNumber) { return firstNumber + nextNumber; }
    },
    MINUS("-") {
        @Override
        public int operate(int firstNumber, int nextNumber) {
            return firstNumber - nextNumber;
        }
    },
    MULTIPLE("*") {
        @Override
        public int operate(int firstNumber, int nextNumber) { return firstNumber * nextNumber; }
    },
    DIVIDE("/") {
        @Override
        public int operate(int firstNumber, int nextNumber) {
            if (nextNumber == 0)
                throw new ArithmeticException();
            return firstNumber / nextNumber;
        }
    };

    private final String value;

    private Operator(String value) {
        this.value = value;
    }

    public int calculate(int firstNumber, int nextNumber) {
        return this.operate(firstNumber, nextNumber);
    }

    public static Operator getOperator(String inputOperator) {
        return (Operator)Arrays.stream(Operator.values())
                .filter(target -> target.value.equals(inputOperator))
                .findFirst()
                .orElseThrow(IllegalArgumentException::new);
    }
}
```

인터페이스를 통해 상수별로 메서드를 다르게 구현했다. 필드로 BinaryOperator를 사용하면 굳이 인터페이스를 구현하지 않고도 같은 기능을 훨씬 짧게 리팩토링할 수 있다. 또한 Enum 클래스는 유용한 메서드들을 제공한다.

* ``values()`` 메소드는 enum안에 존재하는 모든 값들을 순서대로 배열로 반환한다.
* ``ordinal()`` 메소드는 enum 상수의 인덱스를 반환한다.
* ``valueOf()`` 메소드는 문자열 이름에 해당하는 상수를 반환한다.

<br>

## 3. 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

> PayrollDay.java

```java
enum PayrollDay {
    MONDAY, TUESDAY, ..., SUNDAY;

    private PayrollDay() {
    }

    int pay(int minutesWorked, int payRate) {
        int payment = 0;
        if (this == SATURDAY || this == SUNDAY) {
           //주말 계산 로직
        } else {
           //평일 계산 로직
        }
        return payment;
    }
}
```

* 휴가 등의 새로운 값을 열거 타입에 추가하려면 이를 이용하는 잔업 수당 계산 메서드 또한 함께 수정되어야 한다.
* 혹은 Operator 예제처럼 상수별로 별도로 메서드를 구현해야 하는데, 이 또한 코드가 장황하고 관리 또한 복잡해진다.

> PayrollDay.java

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

열거 타입의 필드로 내부 전략 열거 타입을 지정한다. 잔업 수당 계산을 전략 열거 타입에 위임함으로써 메서드 내부 조건문이나 상수별 메서드 구현이 필요 없어진다. 다소 복잡하더라도 메서드가 분리되끼 때문에 더 유연한 패턴이다.

<br>

## 4. 요약

* 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
* 열거 타입은 정의된 상수 개수가 고정 불변일 필요가 없다.
  * 상수가 추가되어도 바이너리 수준에서 호환된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
