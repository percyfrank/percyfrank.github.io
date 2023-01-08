---
title: "[Effective Java] Item 50. 적시에 방어적 복사본을 만들라"
excerpt: "클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 가급적 방어적으로 복사해야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-07
last_modified_at: 2021-02-07
---

## 1. 방어적 복사

Java로 작성한 클래스는 (별도의 노력없이 대체로) 그 불변식이 지켜진다. 그러나 클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍하라. 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 가급적 방어적으로 복사해야 한다.

> Period.java

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }
}
```

> Attack.java

```java
Date start = new Date() ;
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부가 수정된다
```

Date 객체는 가변이기 때문에 Period 객체 생성 시점에 유효성 검사를 하더라도, Period 불변식이 깨질 수 있다. Date는 낡은 API이기 때문에 불변 인스턴스(LocalDateTime 등)을 이용하라.

> Period.java

```java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());
    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
}
```

외부 공격으로부터 Period 인스턴스 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사해야 한다.

* 방어적 복사본을 만든 뒤 유효성 검사를 진행한 점에 주목하자.
  * 멀티스레딩 환경이라면 원본 객체의 유효성을 검사한 후 복사본을 만드는 찰나의 순간에 다른 스레드가 원본 객체를 수정할 수 있기 때문이다.
* Date의 ``clone()`` 메서드를 사용하지 않은 점에 주목하자.
  * Date는 final이 아니므로 ``clone()``이 Date가 정의한 게 아닐 수 있다.
    * 즉, ``clone()``이 악의를 가진 하위 클래스의 인스턴스를 반환할 수도 있다.
  * 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 ``clone()``을 쓰지 않는다.

> Period.java

```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

필드가 가변이기 때문에 제공자가 직접 접근을 허용하면 불변식이 깨질 수 있다. 가변 필드의 방어적 복사본을 만들어 반환한다.

* 접근자 메서드의 방어적 복사에는 ``clone()``을 사용해도 된다.
  * Date 객체가 java.util.Date임이 확실하기 때문이다.

<br>

## 2. 생략 가능 경우

되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다. 방어적 복사에는 성능 저하가 따르고, 항상 쓸 수 있는 것도 아니다.

* 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하거나, 객체의 통제권 이전임이 명백하다면 방어적 복사를 생략할 수 있다.
  * 메서드나 생성자 등 호출자에 더이상 매개변수나 객체 등을 수정하지 말아야 함과 그에 따른 클라이언트의 책임 등을 문서화한다.
* 통제권을 넘겨받기로 한 메서드나 생성자를 가진 클래스들은 악의적인 클라이언트의 공격에 취약하다.
  * 상호 신뢰할 수 있을때, 혹은 불변식이 깨지더라도 그 영향이 호출 클라이언트에 국한될 때만 방어적 복사 생략을 한정한다.
    * 래퍼 클래스 등.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
