---
title: "[Effective Java] Item 77. 예외를 무시하지 말라"
excerpt: "예외는 문제 상황에 잘 대처하기 위해 존재하는데 catch 블록을 비워두면 예외가 존재할 이유가 없어진다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 예외 무시

> WrongPattern.java

```java
try {
    ...
} catch (SomeException e) {
}
```

검사 예외는 호출자로 하여금 적절한 조치를 해달라는 의미이다. 문제 상황을 대처하기 위해 예외가 존재하는데, catch 블록을 비워두면 존재의 이유가 퇴색된다. 비검사 예외 또한 같은 규칙이 적용된다. 예외를 무시하면 프로그램이 오류를 내재한 채 동작하다가 다른 곳에서 죽어버릴 수 있다.

* FileInputStream은 읽기 전용이기 때문에 닫을 때 예외를 무시해도 된다.
  * 파일의 상태를 변경하지 않았으니 복구할 것이 없다.
  * 필요한 정보를 다 읽었으므로 스트림을 닫을 때 남은 작업을 중단할 이유가 없다.
* 혹시나 같은 예외가 자주 발생한다면 **로그**라도 남겨두면 추후 조사에 도움이 된다.

> ExceptionIgnoring.java

```java
try {
    ...
} catch (TimeOutException | SomeException ignored) {
}
```

* 예외를 무시하기로 했다면 catch 블록에 결정 이유를 주석으로 남기고, 예외 변수의 이름도 **ignored**로 바꿔놓도록 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
