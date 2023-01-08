---
title: "[Effective Java] Item 20. 추상 클래스보다는 인터페이스를 우선하라"
excerpt: "다중 구현용 타입으로는 인터페이스가 가장 적합하다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-27
last_modified_at: 2021-01-27
---

## 1. 다중 구현 메커니즘

Java가 제공하는 다중 구현 메커니즘은 추상 클래스와 인터페이스가 있다. 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

### 1.1. 추상 클래스

추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 **하위 클래스**가 되어야 하는 제약이 있다.

* 추상 클래스를 확장한 클래스는 다른 클래스를 상속받을 수 없게 된다.
* 기존 클래스 위에 새로운 추상 클래스를 끼워넣기 어렵다.
  * 두 클래스가 같은 추상 클래스를 확장하고자 하는 경우?
    * 해당 추상 클래스는 두 클래스의 공통 조상이어야 한다.
    * 그렇지 않으면 계층 구조에 혼란을 일으킨다.

### 1.2. 인터페이스

인터페이스가 선언한 메서드를 모두 정의한 클래스는 상속 여부와 상관없이 같은 타입이 된다. 기존 클래스들도 새로운 인터페이스를 쉽게 구현할 수 있다. 또한 인터페이스는 믹스인 정의에 유용하다.

* 믹스인(Mix-In)이란 클래스가 구현할 수 있는 타입이다.
* 대상 타입의 주된 기능에 선택적 기능을 혼합하여 제공한다고 선언하는 효과를 준다.
  * Comparable은 자신을 구현한 클래스의 인스턴스들은 순서를 정할 수 있음을 선언한다.
* 추상 클래스는 믹스인을 정의할 수 없다.

현실에는 계층을 엄격하게 구분하기 어려운 개념들이 많다. 인터페이스로는 계층 구조가 없는 타입 프레임워크를 만들 수 있다.

* 작곡도 하는 가수 클래스는 Singer 인터페이스와 SongWriter 인터페이스 모두를 구현하면 된다.
  * 혹은 Singer와 SongWriter 인터페이스를 모두 확장하고 새로운 메서드까지 추가한 SingerSongWriter 인터페이스를 정의한다.
* 같은 구조를 클래스의 조합으로 구현하려면 계층구조가 복잡해진다.
  * Java는 클래스의 다중 상속을 지원하지 않기 때문이다.

인터페이스의 메서드 중 구현 방법이 명백한 것은 디폴트 메서드로 제공할 수 있다. 이 때, 상속하려는 사람을 위해 Java Doc 문서(@impleSpec)를 남겨야 한다.

* equals와 hashCode 등의 Object 메서드를 디폴트 메서드로 제공해서는 안 된다.
* 타인이 제작한 인터페이스에는 디폴트 메서드를 추가할 수 없다.

<br>

## 2. 템플릿 메서드 패턴

인터페이스와 추상 골격 구현(Skeletal Implementation) 클래스를 함께 제공하는 방법이다.

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

* 인터페이스로 타입을 정하고 골격 구현 클래스가 나머지 메서드들까지 구현한다.
* 단순히 골격 구현 클래스를 확장하는 것만으로 인터페이스를 구현하는 일이 대부분 완료된다.
* 인터페이스와 추상 클래스의 장점을 모두 갖는다.
  * 추상 클래스처럼 구현을 도와주는 동시에 추상 클래스로 타입을 정의할 때의 제약에서 자유롭다.
* 관례상 인터페이스 이름이 XXX라면 추상 골격 구현 클래스는 AbstractXXX이다.

### 2.1. 골격 구현 작성

> AbstractMapEntry.java

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
        //Map.Entry의 기반 메서드를 호출하고 있다.
    }
}
```
* 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드들을 선정한다.
  * 이 기반 메서드들은 골격 구현에서는 추상 메서드가 된다.
* 기반 메서드들을 통해 직접 구현할 수 있는 메서드는 디폴트 메서드로 제공한다.
* 기반 메서드나 디폴트 메서드로 만들지 못한 메서드들은 인터페이스를 구현하는 골격 구현 클래스를 만들어 작성해 넣는다.
  * 골격 구현 클래스는 필요하다면 public 필드와 메서드를 추가해도 된다.

### 2.2. 정리

복잡한 인터페이스는 구현 수고를 덜어주는 골격 구현과 함께 제공할 것을 고려하라. 골격 구현은 가능한 한 인터페이스의 디폴트 메서드로 제공해 해당 인터페이스를 구현한 모든 곳에서 활용하도록 한다. 인터페이스에 걸려있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 많기 때문이다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
