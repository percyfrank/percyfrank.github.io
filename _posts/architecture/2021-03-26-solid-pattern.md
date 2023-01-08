---
title: "객체지향 설계 5대 원칙 : SOLID"
excerpt: "시스템의 각 모듈은 응집도가 높고 결합도가 낮아야 한다."
categories:
  - Architecture
tags:
  - Architecture
date: 2021-03-26
last_modified_at: 2021-03-26
---

## 1. 단일 책임 원칙(Single Responsiblity Principle)

> 어떤 클래스를 변경해야 하는 이유는 오직 하나뿐 이어야 한다.

설계를 잘한 시스템은 요구사항 변경에도 영향을 받는 부분이 적다. 이는 다시 말해 시스템의 한 부분에 변경이 발생했을 때, 그 파급 효과가 시스템 전체로 퍼져나가는 정도가 낮다는 의미다. 따라서 시스템의 각 모듈은 응집도가 높고 결합도가 낮아야 쉽고 유연하게 설계를 변경할 수 있다. 이를 위해 클래스 및 메서드 등 소프트웨어의 부품은 단 하나의 책임(기능)만을 가져야 한다. 한 클래스에서 수행할 수 있는 책임(기능)이 많아지면 클래스 내부-외부의 요소들과 강한 결합이 발생할 가능성이 높고, 이는 유지보수에 악영향을 준다.

### 1.1. 응집도(Cohesion)

응집도란 모듈에 포함된 내부 요소들이 하나의 책임 혹은 관심사에 얼마나 일관되게 집중되어 있는 정도를 나타내는 척도다. 응집도가 낮다면 모듈 내 서로 상관 없는 기능들이 묶여있다고 추측할 수 있다. 또한 변화가 발생했을 때 응집도가 낮으면 모듈에서 변화되는 부분도 작다. 모듈에서 어떤 부분이 변경되어야 하는지, 또한 해당 변경이 다른 부분에 어떤 영향을 끼치는지 번거롭게 확인하는 작업이 수반된다.

### 1.2. 결합도(Coupling)

의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많은 정보를 갖고 있는지를 나타내는 척도다. 한 모듈이 변경되기 위해서 다른 모듈의 변경을 요구하는 정도라고 볼 수 있다. 결합도가 높다면 한 모듈에 변경이 발생할 때 의존하는 모듈들 또한 파급 효과로 인해 함께 변경될 여지가 크다. 인터페이스를 통해 여러 모듈간 느슨하게 연결되어 있으면 확장에 유리하다.

<br>

## 2. 개방-폐쇄 원칙(Open-Closed Principle)

기존의 코드를 변경하지 않고도(Closed) 기능을 추가하거나 수정할 수 있도록(Open) 설계해야 한다. 다시 말해 기능을 변경하거나 확장하면서도 해당 기능을 사용하는 코드는 수정하지 않아야 한다.

> MoviePlayer.java

```java
public class MoviePlayer {

    public void play() {
        System.out.println("Play AVI");
    }

    public static void main(String[] args) {
        MoviePlayer moviePlayer = new MoviePlayer();
        moviePlayer.play();
    }
}
```

* MoviePlayer 인스턴스가 AVI가 아닌 다른 포맷의 파일을 실행하기 위해서는 해당 메서드가 수정되어야 한다.

> MoviePlayer.java

```java

public class MoviePlayer {

    private final Runnable runnable;

    public MoviePlayer(Runnable runnable) {
        this.runnable = runnable;
    }

    public void play() {
        runnable.run();
    }

    public static void main(String[] args) {
        MoviePlayer aviPlayer = new MoviePlayer(() -> System.out.println("Play AVI"));
        MoviePlayer wmvPlayer = new MoviePlayer(() -> System.out.println("Play WMV"));
        aviPlayer.play();
        wmvPlayer.play();
    }
}
```

* 전략 패턴과 같은 인터페이스를 통해 클래스의 변경 없이 새로운 동영상 포맷의 재생이라는 기능을 추가할 수 있다.

<br>

## 3. 리스코프 치환 원칙(Liskov Substitution Principle)

서브타입 클래스는 슈퍼타입 클래스를 언제나 대체할 수 있어야 한다. 즉, 코드 상에서 부모 클래스의 인스턴스를 사용되는 자리에 자식 클래스의 인스턴스를 사용하더라도 계획대로 잘 동작해야 한다. 상속이란 일반화 관계(IS-A)가 성립해야 하고 일관성이 있어야 한다. 자식 클래스는 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 해야 LSP를 만족한다.

``instanceof``로 구체 클래스의 타입을 확인해야 한다면, 이는 서브타입이 슈퍼타입을 대체하지 못함을 시사한다. LSP를 위배하면 확장성(OCP)이 떨어진다. 요구사항이 변경되고 새로운 하위 클래스가 추가될수록 상위 타입을 사용하는 곳에서 코드 수정이 빈번하게 발생할 수 있기 때문이다. 또한 하위 클래스가 상위 타입의 명세에 벗어난 기능을 하면 원칙을 위배한 것이다.

<br>

## 4. 인터페이스 분리 원칙(Interface Segregation Principle)

한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다. 하나의 일반적인 인터페이스보다는 여러 개의 구체적인 인터페이스가 낫다. 클라이언트는 자신이 사용하지 않는 기능에 영향을 받지 않아야 한다.

> AllInOneMachine.java

```java
public class AllInOneMachine implements MultiMachine {

    @Override
    public void print() {
    }

    @Override
    public void fax() {
    }

    @Override
    public void copy() {
    }
}
```

* 복합기 인터페이스를 구현한 복합기는 출력과 팩스 및 복사 등의 기능을 제공한다.
* 출력을 사용하는 클라이언트는 팩스나 복사 기능 변경에 영향을 받지 않아야 한다.
* 클래스와 인터페이스가 비대해진 상태이며 이는 SRP를 위배한다.
  * 사용하지 않는 메서드들도 구현해야 하는 문제가 발생한다.

> AllInOneMachine.java

```java
public class AllInOneMachine implements Fax, Printer, CopyMachine {

    @Override
    public void print() {
    }

    @Override
    public void fax() {
    }

    @Override
    public void copy() {
    }
}
```

* SRP에 따라 단일 책임을 갖는 여러 클래스 및 인터페이스로 분리함으로써 ISP를 만족한다.
* 독립된 인터페이스 구현을 통해 서로에게 영향을 받지 않는 설계를 실현할 수 있다.
  * 이러한 소프트웨어는 시스템의 내부 의존성을 약화시킨다.

<br>

## 5. 의존 역전 원칙(Dependency Inversion Principle)

의존 관계를 맺을 때, 변화하기 쉬운것 보단 변화하기 어려운 것에 의존해야 한다. 변화하기 쉬운 것이란 구체 클래스이며 변화하기 어려운 것은 추상 클래스나 인터페이스다.

> MoviePlayer.java

```java
public class MoviePlayer {

    private final AVIAlgorithm aviAlgorithm;

    public MoviePlayer(AVIAlgorithm aviAlgorithm) {
        this.aviAlgorithm = aviAlgorithm;
    }
}
```

* AVIAlgorithm이라는 구체 클래스를 의존하게 된다면, 다른 동영상 포맷의 알고리즘을 사용하려 할 때 코드에 변화가 생기기 마련이다.


> MoviePlayer.java

```java
public class MoviePlayer {

    private final PlayAlgorithm playAlgorithm;

    public MoviePlayer(PlayAlgorithm playAlgorithm) {
        this.playAlgorithm = playAlgorithm;
    }
}
```

* 동영상 재생 알고리즘이라는 상위 인터페이스에 의존하면 다양한 포맷의 알고리즘을 주입받을 수 있다.

<br>

---

## References

* [SOLID 원칙](https://dev-momo.tistory.com/entry/SOLID-%EC%9B%90%EC%B9%99)
