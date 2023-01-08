---
title: "[Effective Java] Item 24. 멤버 클래스는 되도록 static으로 만들라"
excerpt: "멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 static을 붙인다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. 중첩 클래스

중첩 클래스란 다른 클래스 내부에서 정의된 클래스이다. 중첩 클래스는 자신을 감싼 외부 클래스에서만 사용되어야 하며, 그 외의 경우 탑 레벨 클래스로 정의되어야 한다.

* 정적 멤버 클래스.
* 비정적 멤버 클래스.
* 익명 클래스.
* 지역 클래스.

첫 번째를 제외한 클래스를 모두 Inner Class라고 지칭한다.

<br>

## 2. 멤버 클래스

### 2.1. 정적 멤버 클래스

정적 멤버 클래스는 외부 클래스의 private 멤버에도 접근할 수 있다. 다른 멤버 필드와 동일한 접근 규칙을 적용받는다.

* private이면 Outer Class만 사용이 가능하다.
* 주로 Outer Class와 함께 사용되면 유용한 public 도우미 클래스로 사용된다.
* 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와와 독립적으로 존재할 수 있을 때 사용한다.

private 정적 멤버 클래스는 Outer Class가 표현하는 객체의 한 부분(구성 요소)을 나타낼 때 쓴다.

* Map 구현체는 키-값 쌍을 표현하는 Entry 객체를 가진다.
  * Entry는 Map과 연관되어 있으나 Entry 메서드는 Map을 사용(참조)하지 않는다.
  * 따라서 Entry를 비정적 멤버 클래스로 사용하는 것은 낭비이기 때문에 정적 멤버 클래스로 표현한다.

### 2.2. 비정적 멤버 클래스

비정적 멤버 클래스의 인스턴스는 Outer Class의 인스턴스와 암묵적으로 연결된다. 따라서 비정적 멤버 클래스의 메서드에서 ``OuterClass.this``를 통해 바깥 인스턴스를 참조할 수 있다.

* 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성이 불가능하다.
* 비정적 멤버 클래스가 인스턴스화될 때 바깥 인스턴스와의 관계가 확립되며 변경 불가능하다.
  * 이러한 관계 정보는 비정적 멤버 클래스의 인스턴스에 저장된다.
  * 메모리 공간을 차지하며 생성 시간도 더 걸린다.

비정적 멤버 클래스는 어댑터를 정의할 때 자주 사용된다. 어떤 클래스의 인스턴스를 감싸 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.

> MySet.java

```java
public class MySet<E> implements AbstractSet<E> {

    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        //...
    }
}
```

* Map 인터페이스 구현체들이 ``keySet()``, ``values()`` 등 자신의 컬렉션 뷰를 구현할 때.
* Set, List 등은 iterator를 구현할 때.

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여 비정적 멤버 클래스로 만든다.

* static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
  * 참조를 저장하기 위한 시간과 공간이 소비된다.
* 숨은 참조로 인해 GC가 바깥 인스턴스를 수거하지 못할 수 있다.

### 2.3. 주의점

멤버 클래스가 공개된 클래스의 protected 혹은 public 멤버라면 정적인지 아닌지가 중요하다. 멤버 클래스 역시 공개 API가 되기 때문에, 향후 릴리즈 때 static을 붙이면 하위 호환성이 깨진다.

<br>

## 3. 익명 클래스

> IntArrays.java

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    return new AbstractList<>() {
        @Override public Integer get(int i) {
            return a[i];
        }

        @Override public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        @Override public int size() {
            return a.length;
        }
    };
}
```

이름이 없는 클래스이다. 최근에는 람다를 더 많이 이용한다.

* 선언한 지점에서만 인스턴스를 만들 수 있다.
  * 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
  * 비정적인 문맥이어도 상수 변수 이외의 정적 멤버를 가질 수 없다.
* instanceof나 클래스 이름 검사가 필요한 작업은 수행할 수 없다.
* 여러 인터페이스를 구현할 수 없다.
  * 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
* 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
* 가독성이 떨어질 수 있다.

<br>

## 4. 지역 클래스

지역 클래스는 지역 변수를 선언할 수 있는 곳이라면 어디서든 선언이 가능하다. 유효 범위 또한 지역 변수와 동일하다.

* 멤버 클래스처럼 이름이 있어 재사용이 가능하다.
* 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
* 정적 멤버는 가질 수 없다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
