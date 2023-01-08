---
title: "[Effective Java] Item 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라"
excerpt: "실행자 프레임워크 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 실행자 프레임워크(Executor Framework)

java.util.concurrent 패키지는 실행자 프레임워크 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.

* 특정 태스크가 완료되기를 기다린다.
* 태스크 모음 중 아무것 하나 혹은 모든 태스크가 완료되기를 기다린다.
* 실행자 서비스가 종료하기를 기다린다.
* 완료된 태스크들의 결과를 차례로 받는다.
* 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다.

정적 팩토리를 이용하여 다른 종류의 실행자 서비스(스레드 풀)을 생성하고, 스레드 풀의 스레드 개수를 조정할 수 있다.

* java.util.concurrent.Executors에서 실행자 대부분을 생성 가능하다.
* 혹은 ThreadPoolExecutor 클래스를 직접 사용한다.
* 가벼운 어플리케이션이라면 ``Executors.newCachedThreadPool``이 낫다.
  * 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행된다.
  * 가용한 스레드가 없으면 새로 하나를 생성한다.
  * 무거운 어플리케이션에는 부적절하니 스레드 개수를 고정한 ``Executors.newFixedThreadPool``이나 ThreadPoolExecutor를 사용한다.

스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행해야 한다. 반면 실행자 프레임워크는 작업 단위와 실행 메커니즘이 분리된다.

> Executor.java

```java
ExecutorService exec = Executors.newSingleThreadExecutor();
exec.execute(runnable);
exec.shutdown();
```

* 작업 단위를 나타내는 핵심 추상 개념은 태스크(Runnable과 Callable)이다.
* 태스크를 수행하는 일반적인 매커니즘이 바로 실행자 서비스다.
* 모듈화와 주입을 통해 변화에 유연하게 대처할 수 있다.

<br>

## 2. fork-join

Java 7 부터 실행자 프레임워크는 fork-join 태스크를 지원한다. fork-join 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다.

* ForkJoinTask 인스턴스는 작은 하위 태스크로 나뉘며, ForkJoinPool을 구성하는 스레드들이 이 태스크를 처리한다.
  * 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 처리한다.
* 튜닝하기는 어렵지만 잘 활용하면 높은 처리량과 낮은 지연시간을 달성한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
