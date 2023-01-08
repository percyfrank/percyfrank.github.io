---
title: "[Effective Java] Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라"
excerpt: "wait와 notify를 직접 사용하는 것을 동시성 '어셈블리 언어'로 프로그래밍하는 것에 비유할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 동시성 유틸리티

java.util.concurrent는 실행자 프레임워크, 동시성 컬렉션, 동기화 장치 등을 제공한다. wait와 notify는 올바르게 사용하기가 까다로우니 고수준 동시성 유틸리티를 사용하자.

<br>

## 2. 동시성 컬렉션

* 동시성 컬렉션은 표준 컬렉션 인터페이스에 동시성을 가미한 고성능 컬렉션이다.
* 동기화를 각자의 내부에서 수행한다.
* 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 성능이 하락한다.
* 여러 메서드를 원자적으로 묶어 호출하는 작업은 불가능하다.
  * 따라서 여러 기본 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메서드들이 추가되었다.
  * 이는 Java 8 부터는 일반 컬렉션 인터페이스에도 디폴트 메서드가 되었다.
    * ``putIfAbsent()`` 등.

> Intern.java

```java
public class Intern {
    private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
    }
}
```

* Collections.synchronizedMap보다는 ConcurrentHashMap을 사용하는게 훨씬 빠르다.

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.

* BlockingQueue의 take 메서드는 큐의 첫 원소를 꺼내지만, 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다.
* 작업 큐(생산자-소비자 큐)로 쓰기 적합하다.
  * 작업 큐는 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태이다.
* ThreadPoolExecutor를 포함한 대부분 실행자 서비스 구현체가 BlockingQueue를 사용한다.

<br>

## 3. 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다. CountDownLatch와 Semaphore 및 Phaser 등이 있다.

> ConcurrentTimer.java

```java
public class ConcurrentTimer {

    private ConcurrentTimer() { } // 인스턴스 생성 불가

    public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done  = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

* CountDownLatch는 일회성 장벽으로서, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
* 생성자로 받은 int 값은 래치의 ``countDown()`` 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.
* ``time()`` 메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다.
  * 그렇지 못하면 메서드가 끝나지 않는 스레드 기아 교착 상태에 빠진다.
* 정밀한 실행 시간 측정은 ``System.currentTimeMillis()``가 아닌 ``System.nanoTime()``을 사용한다.

<br>

## 4. wait & notify

> StandardWait.java

```java
synchronized(obj) {
    while (<조건이 충족되지 않는 경우>)
        obj.wait();

      ... //조건이 충족됐을 때의 동작을 수행한다.
}
```

* ``wait()`` 메서드는 스레드가 조건이 충족되기를 기다리게 할 때 사용한다.
* 객체를 잠근 동기화 영역 안에서 호출해야 하며, 대기 반복문 밖에서는 호출하면 안 된다.
  * 반복문은 ``wait()`` 호출 전후로 조건이 만족하는지를 검사한다.
  * 조건이 충족되었는데 스레드가 ``notify()``를 먼저 호출한 후 대기 상태에 빠지면 다시 깨운다는 보장이 없다.
* 안전 실패를 막기 위해 대기 후에 조건을 검사하여 조건이 충족되지 않았으면 다시 대기하게 한다.
* 조건이 만족되지 않아도 스레드가 깨어날 수 있는 몇 가지 상황이 있다.
  * 스레드가 ``notify()``를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
  * 조건이 만족되지 않았음에도 다른 스레드가 실수 혹은 악의적으로 ``notify()``를 호출한다.
    * 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 ``wait()``은 모두 이 문제에 영향을 받는다.
  * 깨우는 스레드가 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 ``notifyAll()``을 호출해 모든 스레드를 깨운다.
  * 대기 중인 스레드가 ``notify()`` 없이도 허위 각성하여 깨어나는 경우가 있다.

일반적으로 ``notifyAll()``을 사용하는 것이 합리적이고 안전하다.

* 다른 스레드까지 깨어나도 기다리던 조건이 충족되었는지 확인하고, 충족되지 않은 경우 다시 대기시키면 큰 영향을 주지 않는다.
* 관련 없는 스레드가 실수로 혹은 악의적으로 ``wait()``를 호출하는 공격으로부터 보호할 수 있다.
  * 그런 스레드가 중요한 ``notify()``를 삼켜버린다면 꼭 깨어나야 할 스레드들이 영원히 대기하게 될 수 있기 때문이다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
