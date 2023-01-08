---
title: "[Effective Java] Item 63. 문자열 연결은 느리니 주의하라"
excerpt: "많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자"
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 문자열 연결 연산자

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐준다. 문자열은 불변이라 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사한다. 따라서 문자열 n개를 잇는 시간은 n^2에 비례한다.

* 성능이 중요하다면 StringBuilder를 사용한다.
  * 결과를 담을 충분한 크기로 초기화해야 한다.
* StringBuffer는 동기화로 인해 스레드 안전하지만 StringBuilder보다는 성능이 약간 떨어질 수 있다.
  * Java 11은 Compact Strings로 인해 일반 문자열 연결 연산도 속도가 많이 개선되었다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
