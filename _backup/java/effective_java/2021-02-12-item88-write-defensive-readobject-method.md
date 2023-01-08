---
title: "[Effective Java] Item 88. readObject 메서드는 방어적으로 작성하라"
excerpt: "readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-12
last_modified_at: 2021-02-12
---

## 1. 숨겨진 생성자

``readObject()`` 메서드는 실질적으로 또 다른 숨겨진 public 생성자이다. 따라서 다른 생성자와 똑같이 주의를 기울여야 한다. 보통의 생성자처럼 인수가 유효한지 검사하고, 필요하다면 매개변수를 방어적으로 복사해야 한다. 이를 수행하지 않으면 공격자가 클래스의 불변식을 쉽게 깨뜨릴 수 있다.

> Period.java

```java
private final Date start;
private final Date end;

private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    if (start.compare(end))
        throw new InvalidObjectException();  
}
```

* ``readObject()``는 바이트 스트림을 매개변수로 받는 생성자라고 생각하면 쉽다.
* 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 발생한다.
  * 정상적인 생성자로 만들어낼 수 없는 객체를 생성할 수 있다.
* 똑같이 불변식을 체크하면 된다.

<br>

## 2. 방어적 복사

정상적인 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.

* 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후, 스트림 끝에 추가된 이 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보(Date)를 얻는다.
* 참조로 얻은 Date를 수정하게 되니, Period는 불변식이 깨진다.
* 해당 인스턴스가 불변이라고 가정하는 클래스에 변경 가능한 인스턴스를 넘기면 큰 보안 문제를 유발한다.

> Period.java

```java
private Date start;
private Date end;

private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    start = new Date(start.getTime());
    end = new Date(end.getTime());

    if (start.compare(end))
        throw new InvalidObjectException();  
}
```

* 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.
* final 필드는 방어적 복사가 불가능하니 이를 제거해서 사용해야 한다.

<br>

## 3. 기타

* 직렬화 프록시 패턴을 통해 유효성 검사 및 방어적 복사 기능을 수행할 수 있다.
* final이 아닌 직렬화 가능 클래스의 ``readObject()`` 메서드는 생성자와 마찬가지로 재정의 가능한 메서드를 호출해서는 안 된다.
  * 규칙을 어겼는데 해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
