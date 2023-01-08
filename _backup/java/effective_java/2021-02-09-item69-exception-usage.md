---
title: "[Effective Java] Item 69. 예외는 진짜 예외 상황에만 사용하라"
excerpt: "예외를 정상적인 제어 흐름을 위해 사용해서는 안 된다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 잘못된 예외 사용

> WrongLoop.java

```java
try {
    int i = 0;
    while (true)
        range[i++].climb();  
} catch (ArrayIndexOutOfBoundsException e) {
}
```

* for-each 문으로 컬렉션을 순회하면 될 것을 예외를 통해 반복문을 종료한다.
* 위 코드는 최적화에 전혀 도움이 되지 않는다.
  * 예외는 예외 상황에 쓸 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다.
    * 최적화에 신경쓰지 않았을 가능성이 크다.
  * try-catch 블록 안에 넣은 코드는 JVM이 적용할 수 있는 최적화가 한계가 있다.
* 반복문에서 다른 버그가 발생했을 때 흐름 제어에 쓰인 예외가 이 버그를 숨겨 디버깅을 어렵게 할 수 있다.

<br>

## 2. 올바른 예외 사용

예외는 오직 예외 상황에서만 써야 한다. 절대로 일상적인 제어 흐름용으로 사용해서는 안 된다. 특히 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야 한다.

> IteratorFoo.java

```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext();) {
    Foo foo = i.next();
}
```

* Iterator 인터페이스는 특정 상태에서만 호출할 수 있는 상태 의존적 메서드를 제공한다.
* 따라서 ``hasNext()`` 같은 상태 검사 메서드도 함께 제공해야 한다.

> IteratorFoo.java

```java
try {
    Iterator<Foo> i = collection.iterator();
    while (true)
        Foo foo = i.next();
} catch (NoSuchElementException e) {
}
```

* 상태 검사 메서드가 없었다면 클라이언트가 위와 같이 예외를 사용해서 체크해야 한다.

### 2.1. 상태 검사 메서드의 대안

* 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인이 상태가 변할 수 있다면 옵셔널이나 특정 값을 사용한다.
  * 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체 상태가 변할 수 있다.
* 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업을 일부 중복 수행한다면 옵셔널이나 특정 값을 선택한다.
* 그 외의 모든 경우에는 상태 검사 메서드가 더 낫다.
  * 가독성이 낫다.
  * 상태 검사 메서드 호출을 잊었다면 상태 의존적 메서드가 예외를 던져 버그를 드러낸다.
  * 반면 특정 값은 검사하지 않고 지나쳐도 발견하기 어렵다.
    * 옵셔널은 예외다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
