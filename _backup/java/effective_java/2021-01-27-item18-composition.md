---
title: "[Effective Java] Item 18. 상속보다는 컴포지션을 사용하라"
excerpt: "상속은 상위 클래스의 API를 결함까지 그대로 승계한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-27
last_modified_at: 2021-01-27
---

## 1. 상속

상속은 코드를 재활용하는 강력한 방법이다. 다음의 경우 상속을 활용해도 안전하다.

* 상위 클래스와 하위 클래스 모두 같은 개발자가 통제하는 패키지에 속해 있다.
* 클래스가 확장할 목적으로 설계되었으며 문서화가 잘 되어있다.

그러나 일반적인 구체 클래스를 패키지 경계를 넘어, 다른 패키지의 구체 클래스를 상속하는 것은 위험하다. (인터페이스 상속은 논외로 친다.) 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

* 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있다.
  * 여파로 코드 한 줄 변경하지 않은 하위 클래스가 오작동할 수 있다.
  * 상위 클래스 변화의 영향이 하위 클래스에게 끼친다.

<br>

## 2. 상속의 단점

> MyHashSet.java

```java
public class MyHashSet<T> extends HashSet<T> {

    public int counts;

    @Override
    public boolean add(T t) {
        counts++;
        return super.add(t);
    }

    @Override
    public boolean addAll(Collection<? extends T> c) {
        counts += c.size();
        return super.addAll(c);
    }
}
```

HashSet을 상속받은 MyHashSet은 잘 동작하지 않는다.

> Main.java

```java
MyHashSet<String> myHashSet = new MyHashSet<>();
myHashSet.addAll(Arrays.asList("1", "2", "3"));
System.out.println(myHashSet.counts); //6
```

결과값 3을 기대했지만 6이 나왔다. MyHashSet은 ``super.addAll()``을 호출하지만, HashSet의 ``addAll()``은 내부적으로 ``add()`` 메서드를 호출한다. 이 때 호출되는 ``add()``는 MyHashSet이 재정의한 메서드이기 때문에 결과값이 이상해진다.

MyHashSet에서 ``addAll()``을 오버라이드하지 않거나 다른 방식으로 오버라이드하면 이러한 문제를 해결할 수 있지만, 상속의 근본적인 문제는 해결할 수 없다.

* 상위 클래스의 메서드 동작을 재정의하는 경우 비용과 더불어서 오류 혹은 성능 저하를 유발할 수 있다.
* 상위 클래스가 자기 메서드를 사용하는 자기 사용 여부는 해당 클래스의 내부 구현 방식이다.
  * 다음 릴리즈때 변경될 수도 있고, 이에 기대면 하위 클래스가 변경에 취약해 깨질 수 있다.
* 하위 클래스에서 의도하지 않은 동작(메서드)이 상위 클래스에 추가될 수 있다.
  * HashTable과 Vector 등이 이로 인해 보안 이슈가 발생했다.
  * 클라이언트가 하위 클래스의 인스턴스로 상위 클래스의 해당 메서드를 사용하는 경우 의도하지 않은 결과가 발생할 수 있다.
    * 하위 클래스는 새로 추가된 상위 메서드를 사용하기 싫은 경우 매번 새롭게 오버라이드 해야 한다.

하위 클래스에 새로운 메서드를 정의하는 경우 다음과 같은 문제가 존재한다.

* 다음 릴리즈 때 상위 클래스에 새로운 메서드가 추가되었는데, 하위 메서드와 시그니쳐가 같고 반환 타입이 다르면 컴파일이 되지 않는다.
* 해당 메서드가 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

<br>

## 3. Composition

기존 클래스를 상속하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스를 참조한다. 새 클래스의 메서드들은 기존 클래스의 대응하는 메서드를 호출한다. 그 결과, 새로운 클래스는 기존 클래스의 내부 구현 방식 및 신규 메서드 추가 등의 영향에서 자유로워진다.

> MySet.java

```java
public class MySet<T> {

    private final Set<T> set;
    public int counts;

    public MySet(Set<T> set) {
        this.set = set;
    }

    public boolean add(T t) {
        counts++;
        return set.add(t);
    }

    public boolean addAll(Collection<? extends T> c) {
        counts += c.size();
        return set.addAll(c);
    }
}
```

> Main.java

```java
MySet<String> mySet = new MySet<>(new HashSet<>());
mySet.addAll(Arrays.asList("1", "2", "3"));
System.out.println(mySet.counts); //3
```

MySet은 생성자로 Set 인터페이스를 받기 때문에 HashSet과 TreeSet 등 다양한 Set 구현체를 사용할 수 있게 된다. 컴포지션을 사용하면 상속으로 인한 문제점이 해결된다. MySet같은 클래스를 **래퍼 클래스**라고 하며, 성능에 크게 영향을 주지 않는다.

<br>

## 4. is-a 및 has-a 관계

> 컴포지션은 특정 클래스의 결함을 숨기는 새로운 API를 설계하지만, 상속은 상위 클래스의 API를 결함까지 그대로 승계한다.  

상속은 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 사용한다.

* 하위 클래스와 상위 클래스의 관계가 is-a인 경우에만 상속한다.
  * 아니라면 상위 클래스는 하위 클래스의 필수 구성요소가 아닌 구현 방법의 하나일 뿐이다.
* is-a 관계라도 상위 클래스가 확장을 위해 설계되지 않았다면 문제가 될 수 있다.

has-a 관계인 컴포지션을 사용해야 할 상황에서 상속을 사용하면 다음과 같은 단점이 있다.

* 내부 구현을 불필요하게 노출한다.
  * API가 내부 구현에 묶이게 되며 클래스의 성능도 제한된다.
* 유사한 이름의 상위 클래스의 메서드가 혼동을 준다.
* 클라이언트가 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 수정할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
