---
title: "[Effective Java] Item 8. finalizer와 cleaner 사용을 피하라"
excerpt: "사용하려거든 불확실성과 성능 저하에 주의해야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. finalizer

Java는 객체 소멸자인 finalizer를 제공하지만 예측 불가능하고 상황에 따라 위험하기 때문에 일반적으로 불필요하다. Java 9부터는 @Deprecated되었지만 여전히 여러 라이브러리에서 사용 중이다.

> C++의 파괴자(destructor)는 특정 객체와 관련된 자원을 회수하는 용도로서 try-with-resources와 비슷할뿐, finalizer와 전혀 다른 개념이다.

### 1.1. 결함

finalizer는 즉시 수행되지 않고 수행 여부조차 보장하지 않으며, GC 알고리즘에 따라 천차만별이다. 제때 실행되어야 하는 자원 해제 작업 및 DB의 공유 자원에 대한 Lock 해제 등 라이프사이클에 상관없이 상태를 영구적으로 수정하는 작업을 finalizer에게 맡기면 안 된다.

* ``System.gc``나 ``System.runFinalization`` 등의 메서드는 실행 가능성을 높여줄뿐 여전히 수행 시점 및 여부를 보장하지 않는다.

finalizer 동작 중 발생한 예외는 무시되며 처리할 작업이 남아있더라도 그 순간 종료된다. 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있으며, 다른 스레드가 해당 훼손된 객체를 사용하면 어떻게 동작할지 예측할 수 없다. 또한 finalizer 동작 중 발생한 예외는 스택 트레이스 등 경고조차 출력하지 않는다.

finalizer는 심각한 성능 하락과 더불어서 finalizer 공격에 노출되어 보안 문제를 야기할 수 있다.

* 생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있다.
  * finalizer가 정적 필드에 자신의 참조를 할당하면 GC가 수집하지 못한다.
* 일그러진 객체를 통해 메서드를 호출함으로써 허용되지 않은 작업을 수행할 수 있다.
  * 생성자 예외 등을 통해 인스턴스화를 막더라도 finalizer가 있다면 뚫릴 수 있다는 의미이다.
* final 클래스들은 하위 클래스를 만들 수 없으니 해당 공격에서 자유롭다.
  * final이 아닌 클래스들은 아무 일도 하지 않는 finalize 메서드를 만들고 final로 명시하면 방어할 수 있다.

<br>

## 2. cleaner

Java 9에서부터 finalizer의 대안으로 제시되었으나 비슷한 단점들로 인해 사용 지양이 권고된다. 다만 finalizer에 비해 cleaner는 자신의 스레드를 통제할 수 있기 때문에 예외 상황 발생시 조금 덜 위험하다.

<br>

## 3. 대안

파일이나 스레드 등 종료해야 할 자원을 담고있는 객체의 경우?

* 클래스에서 AutoCloseable을 구현해주고 인스턴스를 다 쓰고나면 ``close()`` 메서드를 호출해준다.
  * 해당 객체가 닫혔음을 필드에 기록해둔다.
  * 객체가 닫힌 뒤 호출되면 IllegalStateException 등을 던져준다.
* 예외 상황을 대비해 try-with-resources 구문을 사용한다.

> 참고 : AutoCloseable을 구현한 객체는 try-with-resources로 관리될 때 ``close()`` 메서드가 자동으로 호출된다.

<br>

## 4. 사용 이유

아래의 두 가지의 경우에 해당되더라도, 불확실성과 성능 저하에 주의하며 finalizer 및 cleaner를 사용해야 한다.

### 4.1. 안전망 역할

자원의 소유자가 차마 ``close()`` 등으로 자원 회수를 하지 않는 경우를 대비한 안전망 역할을 한다.

* FileInputStream, FileOutputStream 등이 안전망 역할의 finalizer를 사용한다.

### 4.2. Native Peer와 연결된 객체

Native Peer란 일반 Java 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 의미한다. GC는 네이티브 객체를 회수하지 못하기 때문에 finalizer 및 cleaner를 사용한다.

* 단 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않는 경우에만 해당한다.
* 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 ``close()``를 사용한다.

<br>

## 5. cleaner 사용 예제

> Room.java

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출한다.
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
```

* State는 cleaner가 청소할 때 수거하는 자원을 정의하고 있다.
* GC가 수거하지 않거나 ``close()``를 호출하지 않는다면 cleaner가 (언젠가) State의 ``run()``을 호출하게 된다.

State 인스턴스가 Room 인스턴스를 참조하는 경우 순환 참조가 발생하여 GC가 Room을 회수해갈 기회가 사라진다. State가 정적이 아닌 중첩(내부) 클래스라면 자동으로 바깥 객체의 참조를 가지기 때문에 정적으로 선언되었다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
