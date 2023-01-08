---
title: "[Effective Java] Item 75. 예외의 상세 메시지에 실패 관련 정보를 담으라"
excerpt: "사후 분석을 위해 실패 순간의 상황을 정확히 포착해서 예외의 상세 메시지에 담아야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 스택 추적

예외가 발생하면 Java 시스템은 스택 추적(Stack Trace) 정보를 자동 출력한다. 스택 추적은 예외 객체의 ``toString()``을 통해 얻는 문자열이다.

* 사후 분석을 위해 실패 순간의 상황을 정확히 포착해서 예외의 상세 메시지에 담아야 한다.
  * 재현하기 어려운 실패라면 에러 메시지가 유일한 정보일 수 있다.
  * 이러한 정보를 통해 실패 원인을 분석할 수 있다.

실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.

* 단, 보안과 관련한 정보는 주의해서 다루어야 한다.
* 에러 메시지는 너무 장황할 필요는 없다.

예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안 된다.

* 최종 사용자에게는 안내 메시지를, 예외 메시지는 가독성보다는 담긴 내용이 중요하다.

> IndexOutOfBoundsException.java

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
    super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
}
```

* 미리 상세 메시지를 만들어 두고 생성자에서 응용할 수 있다.
* 상세 메시지를 만들어내는 코드가 예외 클래스로 캡슐화되기 때문에, 클래스 사용자는 메시지를 만드는 작업을 중복하지 않아도 된다.
* 예외는 실패와 관련한 정보를 얻을 수 있는 접근자 메서드를 적절히 제공하는 것이 좋다.
  * 포착한 실패는 예외 상황을 복구하는데 유용하므로 비검사보다는 검사 예외에서 더욱 유용하다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
