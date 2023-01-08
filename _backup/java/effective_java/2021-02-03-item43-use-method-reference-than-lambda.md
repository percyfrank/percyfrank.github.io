---
title: "[Effective Java] Item 43. 람다보다는 메서드 참조를 사용하라"
excerpt: "메서드 참조는 람다의 간단명료한 대안이 될 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-03
last_modified_at: 2021-02-03
---

## 1. 메서드 참조(Method Reference)

메서드 참조는 함수 객체를 람다보다도 더 간결하게 만들 수 있다. 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용한다.

> MapTest.java

```java
map.merge(key, 1, (count, incr) -> count + incr);
map.merge(key, 1, Integer::parseInt);
```

* ``merge()`` 메서드는 주어진 키가 맵에 없다면 키, 값을 저장한다.
* 주어진 키가 맵에 존재하는 경우 인자로 주어진 함수를 현재 값과 주어진 값에 적용하여 저장한다.
* 매개변수 수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어난다.
* 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 해당 메서드를 사용한다.
  * 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 설명을 문서화할 수 있다.
* 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다.

### 1.1. 예외

> GoshThisClassNameIsHumongous.java

```java
service.execute(GoshThisClassNameIsHumongous::action);
service.execute(() -> action());
```

메서드 참조 쪽이 항상 짧고 명확한 것은 아니다. ``Function.identity()`` 보다 ``(x -> x)``가 더 짧고 명료하다.

### 1.2. 유형

* 정적 메서드 참조.
  * ``str -> Integer.parseInt(str)``을 ``Integer::parseInt``로 치환한다.
* 한정적 인스턴스 메서드 참조.
  * 참조 대상 인스턴스를 특정한다.
  * 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 동일하다.
  * ``Instance then = Instance.now();``
  * ``t -> then.isAfter(t)``를 ``Instance.now()::isAfter``로 치환한다.
* 비한정적 인스턴스 메서드 참조.
  * 참조 대상 인스턴스를 특정하지 않는다.
  * 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
  * ``str -> str.toLowerCase()``를 ``String::toLowerCase``로 치환한다.
* 클래스 생성자.
  * ``() -> new TreeMap<K,V>``를 ``TreeMap<K,V>::new``로 치환한다.
* 배열 생성자.
  * ``len -> new int[len]``을 ``int[]::new``로 치환한다.
* 생성자 참조는 팩터리 객체로 사용된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
