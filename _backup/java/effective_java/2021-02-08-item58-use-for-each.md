---
title: "[Effective Java] Item 58. 전통적인 for 문보다는 for-each 문을 사용하라"
excerpt: "for-each 문은 명료하고, 유연하고, 버그를 예방해준다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. 전통적인 for 문

반복자와 인덱스 변수를 사용하는 전통적인 for 문은 while 문보다는 낫지만 코드를 지저분하게 만든다. 반복문 내부에서 반복자나 인덱스를 자주 사용하게 되면서 에러 발생 가능성이 높아진다.

* 컴파일러가 해당 에러를 잡아준다는 보장이 없다.
* 컬렉션이냐 배열이냐에 따라 코드 형태가 상이해진다.

<br>

## 2. for-each 문

> Element.java

```java
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next())); //잘못된 사용

for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

향상된 for 문(enhanced for statement)는 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔해진다. 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어 어떤 컨테이너든 상관없다. 직접 반복자나 인덱스를 사용하지 않으니 에러 발생 가능성이 낮아진다.

for-each 문은 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다. 또한 성능 저하도 없다.

### 2.1. 예외

다음의 경우 for-each 문을 사용할 수 없다.

* 파괴적인 필터링 : 컬렉션을 순회하며 선택된 원소를 제거해야 한다면 반복자의 ``remove()``를 호출해야 한다.
  * Collection의 ``removeIf()``를 사용하면 명시적 순회를 피할 수 있다.
* 변형 : 리스트나 배열을 순회하면서 원소의 값 일부 혹은 전체를 교체하는 경우 리스트의 반복자나 배열의 인덱스가 필요하다.
* 병렬 반복 : 여러 컬렉션을 병렬로 순회하는 경우 각각의 반복자나 인덱스 변수를 사용하게 된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
