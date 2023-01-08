---
title: "[Effective Java] Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다"
excerpt: "컬렉션 인터페이스는 반복과 스트림을 동시에 지원한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-05
last_modified_at: 2021-02-05
---

## 1. 스트림의 단점

일반적으로 원소 시퀀스를 반환하는 메서드들은 반환 타입으로 컬렉션 인터페이스나 Iterable 혹은 배열을 사용한다. 기본적으로 컬렉션 인터페이스를 사용한다. for-each 문에서만 쓰이거나 반환된 원소 시퀀스가 컬렉션 관련 메서드를 구현할 수 없을 때 Iterable 인터페이스를 사용한다. 반환 원소 타입이 기본 타입이거나 성능에 민감한 경우 배열을 사용한다.

> Error.java

```java
for (ProcessHandle ph : ProcessHandle::allProcess()::iterator) {
} //타입 추론 한계로 컴파일되지 않는다.

for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle::allProcess()::iterator) {
} //작동은 하지만 쓰게이는 너무 난잡하고 직관성이 떨어진다.
```

* 스트림은 반복을 지원하지 않는다.
* Stream 인터페이스는 Iterable 인터페이스의 추상 메서드를 전부 포함하며 Iterable이 정의한 방식대로 동작한다.
  * 그러나 Stream은 Iterable을 확장하지 않아 for-each로 반복 사용할 수 없다.
* Stream의 ``iterator()`` 메서드에 메서드 참조를 건네면 반복문에서 작동은 하지만, 명시적 형변환이 필요해진다.

> Adapters.java

```java
public class Adapters {

    public static <E> Iterable<E> iterableOf(Stream<E> stream) {
        return stream::iterator;
    }
}
```

> Work.java

```java
for (ProcessHandle p : iterableOf(ProcessHandle.allProcess())) {
}
```

* Stream\<E\>를 Iterable\<E\>로 중개해주는 어댑터 패턴을 사용하면 스트림도 for-each문으로 반복할 수 있다.
* 가독성 측면에서 나쁘며 성능이 악화될 수 있다.

<br>

## 2. 컬렉션 반환 타입

특정 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 스트림을 반환하면 된다. 그러나 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환한다. 공개 API를 작성할 때는 두 방식을 사용하는 사람 모두를 배려해야 한다.

* Collection 인터페이스는 Iterable의 하위 타입이고 ``stream()`` 메서드도 제공한다.
  * 반복과 스트림을 동시에 지원한다.
* 원소 시퀀스를 반환하는 공개 API의 반환 타입은 Collection이나 그 하위 타입을 사용하는 것이 최선이다.
* Arrays 역시 관련 메서드로 손쉽게 반복과 스트림을 지원할 수 있다.
  * 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환해도 최선일 수 있다.
* 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안 된다.

### 2.1. 전용 컬렉션 구현

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 검토한다.

> PowerSet.java

```java
public class PowerSet {

    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }
}
```

* 멱집합(한 집합의 모든 부분집합을 원소로 하는 집합)을 반환하는 경우이다.
  * 원소가 n개면 멱집합의 원소 개수는 2^n이 된다.
* 멱집합을 표준 컬렉션 구현체에 저장하지 말고 AbstractList를 통해 전용 컬렉션을 구현한다.
* Collection의 ``size()`` 메서드는 int값을 반환하기 때문에 최대값은 2^31 - 1로 제한된다.
  * 입력 집합 원소 수가 30을 넘으면 예외를 던진다.
* 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다.
* AbstractCollection을 활용해서 컬렉션 구현체를 작성할 때 Iterable용 메서드와 ``size()`` 및 ``contains()``를 구현하면 된다.
  * 구현이 불가능한 경우 스트림이나 Iterable을 반환하는 것이 낫다.
  * 원한다면 별도의 메서드를 두어 두 방식을 모두 제공해도 된다.

### 2.2. 리스트 예제

때로는 단순히 구현하기 쉬운 쪽을 선택하기도 한다.

> SubLists.java

```java
public class SubLists {

    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```

* 입력 리스트의 부분 리스트를 반환할 때 모든 부분 리스트를 스트림으로 반환한다.
* ``flatMap()``을 통해 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다.

> SubLists.java

```java
public static <E> Stream<List<E>> of(List<E> list) {
    return IntStream.range(0, list.size())
            .mapToObj(start ->
                    IntStream.rangeClosed(start + 1, list.size())
                            .mapToObj(end -> list.subList(start, end)))
            .flatMap(x -> x);
}
```

* 프레픽스와 서픽스를 통해 부분 리스트를 얻는 방식은 2중 반복문과 로직이 동일하다.
* 2중 반복문 또한 Stream을 통해 구현할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
