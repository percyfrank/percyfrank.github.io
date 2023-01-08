---
title: "[Modern Java in Action] 2장. 동작 파라미터화 코드 전달하기"
excerpt: "동적 파라미터화는 빈번한 요구사항 변경에 대처할 수 있는 개발 패턴이다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-02-19
last_modified_at: 2021-02-19
---

## 1. 동적 파라미터화(Behavior Parameterization)

동적 파라미터화는 빈번한 요구사항 변경에 대처할 수 있는 개발 패턴이다. 코드 블록을 다른 메서드의 인수로 전달할 수 있다.

> FilteringApples.java

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor() == color) {
            result.add(apple);
        }
    }
    return result;
}
```

* 사과 리스트를 필터링하는데 다른 기준을 적용하고 싶다면 매번 똑같은 로직을 가진 메서드를 새로 작성해야 한다.
  * 반복은 죄악이다.

> ApplePredicate.java

```java
interface ApplePredicate {
    boolean test(Apple a);
}
```

> AppleWeightPredicate.java

```java
class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}
```

> AppleColorPredicate.java

```java
class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return apple.geColor().equals(RED);
    }
}
```

> FilteringApples.java

```java
public static List<Apple> filter(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

> Main.java

```java
List<Apple> result = filter(list, new AppleWeightPredicate());
List<Apple> result2 = filter(list, new AppleColorPredicate());
```

* 알고리즘을 정의하고 캡슐화한 다음 런타임 시점에 선택하는 전략 패턴과 유사하다.
* 동적 파라미터화는 추상화된 동작(메서드)을 인자로 제공함으로써 하나의 메서드 시그니쳐로 다양한 동작을 수행할 수 있다.
  * 컬렉션을 순회하는 로직과 각 원소들을 검증하는 로직을 분리하게 된다.

<br>

## 2. 리팩토링

위의 ``filter()`` 메서드에 새로운 필터링 기준을 사용하고 싶으면, 매번 ApplePredicate 인터페이스를 구현한 클래스를 새로 생성해야 한다. 일회성 동작의 성격을 지닌 클래스를 작성하고 파일로 관리하는 것은 다소 번거롭다.

### 2.1. 익명 클래스

익명 클래스는 클래스의 선언과 인스턴스화를 동시에 수행한다.

> FilteringApples.java

```java
List<Apple> redApples2 = filter(inventory, new ApplePredicate() {
    @Override
    public boolean test(Apple a) {
        return a.getColor() == Color.RED;
    }
});
```

* Swing과 같은 GUI 어플리케이션에서 콜백 이벤트 리스너에 많이 사용된다.
* 여전히 객체를 생성하고 명시적으로 메서드를 구현하는 등의 작업이 수반된다.

### 2.2. 람다 표현식

> Filtering.java

```java
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList();
    for(T t : list) {
        if (p.test(t)) {
            result.add(t);
        }
    }
    return result;
}
```

> Main.java

```java
List<Apple> apples = filter(list, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> numbers = filter(list2, (Integer i) -> i % 2 == 0);
```

* 람다 표현식을 사용하면 코드가 더 간결해지며, 지네릭스를 더해 더 다양한 타입을 지원하도록 추상화했다.

<br>

## 3. 실사용 사례

* 컬렉션 API의 ``sort()`` 메서드는 인자로 Comparator 인터페이스를 받는다.
  * 람다 표현식이나 Comparator에서 제공하는 ``comparing()`` 등의 디폴트 메서드로 비교 기준을 동적 파라미터화한다.
* 스레드 API는 수행할 동작을 Runnable 인터페이스로 받는다.
  * ``new Thread(() -> System.out.println("hi"));``
* 실행자 서비스(ExecutorService)는 생성한 스레드 풀에 작업을 전달하고 그 결과를 Future에 저장할 수 있다.
  * 메서드 인자로 Callable 인터페이스를 받는다.
  * ``ExecutorService.submit(() -> Thread.currentThread().getName())``
* Swing 등 GUI 어플리케이션은 특정 이벤트(클릭 등)에 발생하는 동작을 메서드 인자로 받는다.
  * EventHandler 인터페이스를 인자로 받아 콜백 이벤트 리스너 기능을 구현한다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
