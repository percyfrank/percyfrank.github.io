---
title: "[Effective Java] Item 46. 스트림에서는 부작용 없는 함수를 사용하라"
excerpt: "스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-04
last_modified_at: 2021-02-04
---

## 1. 순수 함수

순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.

* 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다.
* 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
  * 이를 위해서는 스트림 연산에 건내는 함수 객체는 모두 부작용이 없어야 한다.

> Freq.java

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}

Map<String, Long> freq= new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

* forEach 연산은 종단 연산 중 기능이 적고 병렬화가 불가능하다.
  * 스트림답지 않아 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 사용하는 것은 지양한다.
* 수집기를 사용하면 위의 코드처럼 부작용없이 빈도표를 초기화 할 수 있다.

<br>

## 2. 수집기

축소(Reduction)란 스트림의 원소들을 객체 하나에 취합한다는 뜻이다. Collectors 클래스는 스트림의 원소를 컬렉션으로 모을 수 있는 수집기를 제공한다.

### 2.1. toMap()

> Converter.java

```java
private static final Map<String, Operation> stringToEnum = Stream.of(values())
            .collect(Collectors.toMap(Object::toString, e->e));
```

* 키에 맵핑하는 함수와 값에 맵핑하는 함수를 인수로 받아 스트림을 Map에 수집한다.

> Hits.java

```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));

//toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

* BinaryOperator\<U\> 형태의 병합 함수까지 제공할 수 있다.
  * U는 해당 맵의 값 타입이다.
* 키를 공유하는 값들은 병합 함수를 이용해 기존 값에 합쳐진다.
  * 혹은 동일한 키에 값들이 충돌하는 경우 마지막 값을 취하는 용도로 사용할 수 있다.
* 네 번째 인수로 Supplier 맵 팩토리를 받을 수 있다.
  * EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 지정할 수 있다.
* ``toConcurrentMap()`` 등의 변종이 존재한다.

### 2.2. groupingBy()

> 자세한 내용은 [[Java] Stream 데이터를 groupingBy로 그룹핑해 Map으로 수집하기](https://xlffm3.github.io/java/map_using_collectors/) 포스팅을 함께 참조하자.

입력으로 분류 함수(classifier)를 받고 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.

> Anagram.java

```java
Map<String, List<String>> result = words.collect(groupingBy(word -> alphabetize(word)));
```

* 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트이다.

> Anagram.java

```java
Map<String, Set<String>> result = words.collect(groupingBy(word -> alphabetize(word), toSet()));

Map<String, Long> result = words.collect(groupingBy(word -> alphabetize(word), TreeMap::new, counting()));
```

* ``groupingBy()``가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면 분류 함수와 함께 다운스트림 수집기를 명시해야 한다.
  * ``toSet()``, ``toCollection(collectionFactory)`` 등.
* ``counting()``을 넣으면 카테고리에 해당하는 원소의 개수를 값으로 맵핑한다.
* Supplier 맵 팩토리를 지정하여 특정 맵 구현체를 지정할 수 있다.
  * mapFactory 매개변수가 downStream 매개변수보다 앞에 놓인다.
  * 점층적 인수 목록 패턴에 어긋난다.
* ``groupingByConcurrent`` 메서드들은 ConcurrentHashMap 인스턴스를 만들어준다.

### 2.3. 기타

기타에 해당하는 메서드들은 Collectors에 정의되어 있으나 수집과는 관련이 없다.

* ``minBy()``와 ``maxBy()``는 인수로 받은 비교자를 이용해서 스트림에서 값이 작은 혹은 가장 큰 원소를 찾아낸다.
  * Stream 인터페이스의 ``min()`` 및 ``max()`` 메서드를 살짝 일반화했다.

> JoinTest.java

```java
List<String> words = new ArrayList<>();
String result = words.stream()
        .collect(Collectors.joining(","));
```

* ``joining()``은 문자열 등의 CharSequence 인스턴스의 스트림에만 적용할 수 있다.
  * 구분 문자를 매개변수로 받아 연결 부위에 삽입한다.
  * Prefix와 Suffix 매개변수도 추가적으로 받아 삽입할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
