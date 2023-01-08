---
title: "[Effective Java] Item 26. 로 타입은 사용하지 말라"
excerpt: "로 타입을 사용하면 제네릭이 제공하는 안정성과 표현력을 모두 잃게 된다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. 제네릭

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 제네릭 클래스 혹은 제네릭 인터페이스라고 한다.

* ``List<E>``의 E가 타입 매개변수이다.
* 제네릭 타입을 정의하면 그에 딸린 로 타입(Raw Type) 또한 정의된다.
  * 로 타입이란 제네릭 타입에서 타입 매개변수를 사용하지 않을 때를 의미한다.
  * ``List<E>``의 로 타입은 ``List``이다.

로 타입은 타입 선언에 제네릭 타입 정보가 전부 지워진 것처럼 동작하는데, 제네릭을 사용하기 이전의 코드와 호환되도록 하기 위해 도입되었다.

<br>

## 2. 로 타입의 단점

> Stamps.java

```java
List stamps = new ArrayList();
stamps.add(new Coin()); //컴파일러가 "unchecked call" 경고를 발생
for (int i = 0; i < stamps.size(); i++) {
    Stamp stamp = stamps.get(i); //ClassCastException 발생
}
```

로 타입 컬렉션에 데이터를 담을 때는 컴파일러가 에러가 아닌 경고만 발생시키기 때문에 컴파일이 정상적으로 완료된다. 런타임 시점에서 컬렉션에서 데이터를 꺼내 형변환할 때 잘못된 캐스팅으로 인해 예외가 호출된다.

오류는 가능한 컴파일 시점에서 발견하는 것이 좋다. 런타임 시점에서 문제가 일어나는 코드는 원인 코드와 물리적으로 떨어져 있기 때문에 디버깅하기 곤란해진다.

> Stamps.java

```java
List<Stamp> stamps = new ArrayList<>();
stamps.add(new Coin()); //컴파일 실패
```

제네릭을 선언하면 컴파일 시점에서 컴파일러가 체크하고 오류를 발생시킨다. 이후 해당 컬렉션에서 데이터를 꺼내는 코드들은 컴파일러가 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장한다.

* 로 타입을 사용하면 제네릭이 제공하는 안정성과 표현력을 모두 잃게 된다.

<br>

## 3. 임의 객체를 허용하는 매개변수화 타입

``List`` 로 타입은 사용하면 안 되지만, ``List<Object>`` 같은 임의 객체를 허용하는 매개변수화 타입은 괜찮다.

> Raw.java

```java
public class Raw {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, "1");
        String s = strings.get(0);
    }

    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }
}
```

* 단, ``List<String>``은 로 타입 ``List``의 하위 타입이지만 ``List<Object>``의 하위 타입은 아니다.
* ``unsafeAdd()`` 메서드가 인자로 ``List`` 로 타입을 받는다면 컴파일되지만, ``List<Object>``는 에러가 난다.

### 3.1. 비한정적 와일드카드 타입

> Raw.java

```java
private static void unsafeAdd(List<?> list, Object o);
```

* 제네릭 타입을 사용하고 싶으나 실제 타입 매개변수를 신경쓰고 싶지 않을 때 물음표를 사용한다.
* 비한정적 와일드카드 타입은 로 타입과 다르게 안전하다.
  * 로 타입 Collection에는 아무 원소나 집어넣기 때문에 타입 불변식이 훼손된다.
* ``Collection<?>``는 null 이외에는 어떤 원소도 집어넣을 수 없고 컴파일러가 체크해준다.
  * 컬렉션에서 꺼낼 수 있는 객체의 타입도 전혀 알 수 없다.

<br>

## 4. 기타 규칙

class 리터럴에는 로 타입을 써야 한다.

* ``List.class``는 허용하지만 ``List<String>.class``는 사용이 불가능하다.

제네릭 타입 정보는 런타임시 삭제된다.

> Test.java

```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```

* instanceof 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에 적용할 수 없다.
* instanceof는 로 타입이든 비한정적 와일드카드 타입이든 동일하게 작동한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
