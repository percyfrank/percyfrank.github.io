---
title: "[Effective Java] Item 6. 불필요한 객체 생성을 피하라"
excerpt: "객체 생성에는 비용이 수반된다는 사실을 명심하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. String

> Main.java

```java
String a = new String("a");
```

문자열에 new를 호출하면 매번 새로운 인스턴스가 생성된다.

> Main.java

```java
String a = "a";
```

상수 문자열을 대입하면 하나의 인스턴스가 생성되지만 JVM의 Heap 내부 Constant Pool에 저장된다. 이후 똑같은 문자열 리터럴을 사용하는 코드들은 String 인스턴스를 새로 생성하지 않고, Constant Pool에 저장된 인스턴스를 재사용한다.

String의 ``matches()`` 메서드는 내부에서 정규표현식용 Pattern 인스턴스를 생성하지만, 한 번 쓰이고 버려지기 때문에 GC의 대상이 된다. 문제는 Pattern이 입력받은 정규표현식에 해당하는 유한 상태 머신을 만들기 때문에 생성 비용이 높다.

따라서 정규표현식으로 문자열을 비교할 때 Pattern 인스턴스를 캐싱해두고 재사용하는 것이 성능상 유리하다.

> 자세한 내용은 [String과 Matcher 및 Pattern의 ``matches()`` 메서드의 내부 코드 구조를 정리해둔 포스팅](https://xlffm3.github.io/woowacourse/Woowacourse_precourse_racing/#4-%EC%BD%94%EB%93%9C%EB%A5%BC-%EA%B9%8C%EB%B3%B4%EB%8A%94-%EC%8A%B5%EA%B4%80--string)을 참조하자.

<br>

## 2. 직관적이지 않은 경우

생성자가 아닌 정적 팩토리 메서드를 사용하면 캐싱된 객체를 참조할 수 있기 때문에 불필요한 객체 생성을 피할 수 있다. 불변 객체는 재사용해도 안전하지만 덜 명확하거나 직관적이지 않은 경우가 있다.

* Map의 ``keySet()`` 메서드는 매번 같은 Set 인스턴스를 반환한다.
* 의도하지 않은 오토박싱(Auto Boxing).
  * 기본 타입과 박싱된 기본 타입간의 자동 상호 변환을 해주는 기술이다.
  * 박싱된 기본 타입 인스턴스(Long, Integer)보다는 기본 타입(long, int)을 사용하는 것이 성능상 유리하다.

DB 연결같이 생성 비용이 비싼 객체는 별도의 Pool을 관리하여 재사용하는 것이 유리하다. 그러나 무겁지 않은 객체들을 재사용하기 위해 자체 객체 풀을 만들 필요는 없다.

* JVM과 GC가 잘 최적화되어 있기 때문에 작은 객체를 생성하고 회수하는 것은 큰 부담이 되지 않는다.
* 오히려 자체 객체 풀이 메모리 사용량을 늘려 성능 저하로 이어진다.

> Item 50. 방어적 복사(Defensive Copy)와 대조적인 개념이니 함께 참조하자.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
* [String Constant Pool in Java](https://www.geeksforgeeks.org/string-constant-pool-in-java)
