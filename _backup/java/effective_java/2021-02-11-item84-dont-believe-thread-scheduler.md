---
title: "[Effective Java] Item 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라"
excerpt: "스레드 스케줄러에 의존하는 프로그램은 견고성과 이식성을 해치게 된다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 스레드 스케줄러

여러 스레드가 실행 중이면 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 정한다. 그러나 프로그램이 이 정책에 좌지우지돼서는 안 된다. 정확성이나 성능이 스레드 스케줄러에 의존하게 되면 다른 플랫폼에 이식하기 어렵다.

* 이식성 좋은 프로그램은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다.
  * 스레드 스케줄러가 고민할 거리가 줄어든다.
  * 실행 가능한 스레드의 수는 전체 스레드 수와 다르다.
* 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만든다.
* 실행 가능한 스레드 수를 적게 유지하는 방법은 각 스레드가 작업을 완료한 후 다음 일거리가 생길 때까지 대기하도록 하는 것이다.
  * 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.
  * 스레드 풀을 적절히 설정해야 한다.

<br>

## 2. 바쁜 대기

> SlowCountDownLatch.java

```java
public class SlowCountDownLatch {
    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                    return;
            }
        }
    }
    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```

* 스레드는 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사하는 바쁜 대기 상태가 되면 안 된다.
* 스케줄러의 변덕에 취약하며 프로세서에 부담을 주어 다른 작업이 실행될 기회를 박탈한다.

<br>

## 3. yield 및 우선순위

특정 스레드가 다른 스레드들에 비해 많은 시간을 얻지 못하더라도 ``Thread.yield()`` 사용은 지양해야 한다. 해당 메서드는 테스트할 수단이 없으며, JVM이나 OS마다 결과가 천차만별일 수 있다.

스레드 우선순위를 조율해서 어플리케이션의 반응 속도를 변경할 수 있겠으나, 이는 Java 어플리케이션의 이식성에 가장 큰 악영향을 끼친다. 차라리 어플리케이션의 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치를 취해야 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
