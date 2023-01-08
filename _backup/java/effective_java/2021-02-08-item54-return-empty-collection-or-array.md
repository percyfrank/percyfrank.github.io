---
title: "[Effective Java] Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라"
excerpt: "null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. null을 반환하는 안티 패턴

컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환한다면, 이를 사용하는 코드들은 항상 ``xxx != null`` 등의 방어 코드를 삽입해야 한다. 그렇지 않으면 NPE가 발생하기 때문이다.

* 빈 컨테이너 할당이 성능 저하의 주범이라고 확인되지 않는 한, 성능 차이는 신경 쓸 수준이 못 된다.
* 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.
* 따라서 null 대신 빈 컬렉션이나 배열을 반환해야 한다.

> Cheese.java

```java
public List<Cheese> getCheeses() {
    return cheeseInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheeseInStock);
}
```

* 만약 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨린다면, 매번 똑같은 빈 '불변' 컬렉션을 반환하면 된다.

> Cheese.java

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheeseInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

* 배열 또한 null이 아닌 길이가 0인 배열을 반환한다.
* 매번 길이가 0인 배열을 할당하는 것이 신경쓰이면 미리 선언해두고 반환한다.
  * 길이 0인 배열은 모두 불변이다.
* 단순히 성능 개선을 목적으로 ``toArray()``에 넘기는 배열을 미리 할당하는 것은 추천하지 않는다.
  * 오히려 성능이 떨어질 수 있다.
  * ``toArray()`` 메서드는 주어진 배열이 충분히 크지 않다면 내부에서 배열을 새로 만들어 원소를 담아 반환하기 때문이다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
