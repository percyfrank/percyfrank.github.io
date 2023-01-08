---
title: "[Effective Java] Item 27. 비검사 경고를 제거하라"
excerpt: "코드의 타입 안전성이 보장된다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. 비검사 경고

* 비검사 형변환 경고.
* 비검사 메서드 호출 경고.
* 비검사 매개변수화 가변인수 타입 경고.
* 비검사 변환 경고.

비검사 경고를 모두 제거한다면 런타임 중 ClassCastException이 발생할 일이 없다. 경고를 제거할 수 없으나 타입이 안전하다고 확신한다면 ``@SuppressWarnings("unchecked")`` 애너테이션을 달아 경고를 숨기면 된다.

* 타입 안전함이 검증되지 않으면 런타임시 예외가 발생할 수 있다.
* 안전하다고 검증된 비검사 경고를 숨기지 않으면 진짜 문제되는 경고를 식별하기 어렵다.
* 해당 애너테이션은 변수, 메서드, 클래스 등 어떤 선언에도 달 수 있다.
  * 가급적 좁은 범위에 적용해야 한다.
  * 항상 안전한 이유 또한 주석으로 문서화해야 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
