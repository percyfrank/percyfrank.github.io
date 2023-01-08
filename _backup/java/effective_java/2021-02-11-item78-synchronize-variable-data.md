---
title: "[Effective Java] Item 78. 공유 중인 가변 데이터는 동기화해 사용하라"
excerpt: "동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 동기화

동기화란 배타적 실행을 지원한다. 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는다.

* 메서드가 상태를 가진 객체에 접근할 때 객체에 락을 건다.
* 락을 건 메서드는 객체의 상태를 확인하고 필요시 수정한다.
* 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.
* 어떤 메서드도 해당 객체의 상태가 일관되지 않은 순간을 볼 수 없다.

즉, 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다. 동기화는 일관성이 깨진 상태를 볼 수 없게 해주며, 동기화된 메서드나 블락에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

### 1.1. 원자적(Atomic)

명세상 long과 double 외의 변수를 읽고 쓰는 동작은 원자적이다. 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴이 보장된다.

* 그러나 한 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장하지 않는다.
* 따라서 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.

<br>

## 2. 동기화 방법

> StopThread.java

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

* 위 프로그램은 1초 후에 종료되지 않는다.
* 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤 보게 될지 보증할 수 없다.

> StopThread.java

```java
//원래 코드
while (!stopRequested)
    i++;

//최적화한 코드  
if (!stopRequested)
    while (true)
        i++;
```

* 동기화가 빠지면 가상 머신이 코드를 Hoisting 기법으로 최적화하기 때문에, 프로그램이 응답 불가 상태가 될 수 있다.

> StopThread.java

```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}  
```

* 쓰기와 읽기 모두가 동기화되어 동작을 보장한다.

> StopThread.java

```java
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

* volatile 선언을 사용하면 매번 동기화하는 비용이 줄어든다.
  * 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

> DangerousVolatile.java

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

* 증가 연산자로 인해 nextSerialNumber 필드에 두 번 접근하게 된다.
  * 값을 읽고, 1을 증가시킨 새로운 값을 변수에 저장한다.
  * 두 번째 스레드가 이 두 접근 사이를 비집고 들어오면 첫 번째 스레드와 동일한 값을 받는다.
  * 스레드 통신은 지원하지만 원자성(배타적 실행)을 지원하지 못했다.
* synchronized 키워드를 붙이면 해결 가능하다.

> AtomicLong.java

```java
private static final AtomicLong nextSerialNumber = new AtomicLong();

public static int generateSerialNumber() {
    return nextSerialNumber.getAndIncrement();
}
```

* java.util.concurrent.atomic 패키지는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들을 제공한다.
  * volatile은 동기화의 두 효과 중 통신 쪽만 지원한다.
  * 해당 패키지는 원자성(배타적 실행)까지 지원한다.

<br>

## 3. 정리

* 가장 좋은 방법은 가변 데이터는 단일 스레드에서만 사용한다.
  * 멀티 스레드 환경에서는 공유하지 않거나 불변 객체만 공유한다.
* 스레드 정책은 문서에 남겨 유지보수 과정에도 지켜지도록 한다.
* 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 객체에서 공유하는 부분만 동기화해도 된다.
  * 해당 객체를 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다.
  * 사실상 불변이며, 이런 객체를 다른 스레드에 건네는 행위를 안전 발행이라고 한다.
* 객체를 안전하게 발행하는 방법들이란?
  * 클래스 초기화 과정에서 객체를 정적, volatile, final 필드 혹은 락을 통해 접근하는 필드에 저장한다.
  * 동시성 컬렉션에 저장한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
