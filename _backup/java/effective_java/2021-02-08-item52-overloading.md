---
title: "[Effective Java] Item 52. 다중정의는 신중히 사용하라"
excerpt: "일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. 다중정의(Overloading)

재정의(Overriding)한 메서드는 동적으로 선택되지만, 다중정의(Overloading)한 메서드는 정적으로 선택된다.

> CollectionClassifier.java

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

* 재정의된 매서드는 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다.
  * 컴파일타임에 그 인스턴스의 타입이 무엇이었냐는 상관없다.
* 다중정의된 메서드 사이에서는 객체의 런타임 타입은 중요치 않다.
  * 선택은 오직 매개변수의 컴파일타임 타입에 의해 이뤄진다.
* CollectionClassifier는 재정의가아닌 다중정의를 잘못 사용함으로써 항상 **그 외**만 출력된다.

### 1.1. 유의사항

API 사용자가 매개변수를 넘기면서 어떤 다중정의 메서드가 호출될지를 모른다면 오작동하기 쉽다. 다중정의가 혼동을 일으키는 상황을 피해야 한다.

* 매개변수 수가 같은 다중정의는 만들지 말자.
  * 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다.
* 다중정의하는 대신 메서드 이름을 다르게 지어줄 수 있다.
  * ``writeInt()``, ``writeLong()``, ``readInt()``, ``readLong()`` 등.

생성자는 재정의할 수 없으며 정적 팩토리라는 대안이 있다. 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우가 있지만, 다음 안전 대책이 존재한다.

* 매개변수 수가 같은 다중정의 메서드가 많더라도, 그중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분되면 헷갈릴 일이 없다.
  * 매개변수 중 하나 이상이 근본적으로 다르면 된다.
  * 근본적으로 다르다는 건 두 타입의 값을 서로 어느 쪽으로든 형변환할 수 없다.

### 1.2. 오토박싱

> SetList.java

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list); //set과 list는 출력 결과가 다르다.
    }
}
```

* 오토박싱이 도입되면서 다중정의에 더욱 주의를 기울여야 한다.
  * 오토박싱과 제네릭의 도입 전에는 Object와 int가 근본적으로 달랐다.
  * 그러나 오토박싱이 등장하면서 근본적으로 다르지 않게 되었다.
* 리스트는 ``remove(int)``와 ``remove(Object)``를 다중정의하고 있다.
  * i 인덱스의 원소 삭제가 아닌 Integer i라는 원소를 삭제하고 싶다면, ``list.remove(Integer.valueOf(i))``를 사용해야 한다.

### 1.3. 함수형 인터페이스

메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다. 서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않기 때문이다.

<br>

## 2. 포워딩

다중정의를 하더라도 여러 메서드들이 동일한 기능을 수행한다면 신경 쓸 게 없다. 가장 일반적인 방법은 상대적으로 더 특수한 다중정의 메서드에서 덜 특수한(더 일반적인) 다중정의 메서드로 일을 포워딩하는 것이다.

> String.java

```java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence) sb);
}
```

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
