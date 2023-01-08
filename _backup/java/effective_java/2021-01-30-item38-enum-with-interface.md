---
title: "[Effective Java] Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라"
excerpt: "클라이언트는 인터페이스를 구현해 자신만의 열거 타입을 만들 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-30
last_modified_at: 2021-01-30
---

## 1. 열거 타입의 확장

일반적으로 열거 타입을 확장하는 것은 바람직하지 않다.

* 확장한 타입의 원소는 기반 타입의 원소로 취급하지만 반대는 성립하지 않는다.
* 기반 타입과 확장된 타입들의 원소 모두 순회할 방법이 마땅치 않다.
* 확장성을 높이면 고려해야 할 요소가 많아진다.

<br>

## 2. 인터페이스 구현

> Operation.java

```java
public interface Operation {
    double apply(double x, double y);
}
```

> BasicOperation.java

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

> ExtendedOperation.java

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

* 클라이언트는 인터페이스를 구현해 자신만의 열거 타입을 만들 수 있다.
* API가 기본 열거 타입을 직접 명시하지 않고 인터페이스 기반으로 작성되었다면?
  * 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.
  * 다형성의 원리이다.
* 열거 타입은 기본적으로 Enum 클래스를 상속받기 때문에 추상 클래스를 상속받을 수 없다.

<br>

## 3. 제네릭 관련 복습

> Test.java

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

* 한정적 타입 토큰?
  * Class 객체는 열거 타입이면서 동시에 Operator의 하위 타입이어야 한다.

> Test.java

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

* 다른 방법으로는 클래스 객체 대신 한정적 와일드카드 타입 컬렉션을 넘기는 방법이다.

<br>

## 4. 단점

> BasicOperation.java

```java
public static BasicOperation getOperator(String inputOperator) {
    return (BasicOperation) Arrays.stream(BasicOperation.values())
            .filter(target -> target.symbol.equals(inputOperator))
            .findFirst()
            .orElseThrow(IllegalArgumentException::new);
}
```

열거 타입끼리 구현을 상속할 수 없다. 연산 기호를 찾는 로직이 BasicOperation 클래스뿐만 아니라 ExtendedOperation 클래스에도 모두 중복되어야 한다.

* 중복량이 많아지면 별도의 정적 유틸리티 클래스로 코드 중복을 없앨 수 있기는 하다.
* 인터페이스에 디폴트 메서드를 추가할 수는 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
