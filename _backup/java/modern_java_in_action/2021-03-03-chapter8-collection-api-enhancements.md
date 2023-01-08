---
title: "[Modern Java in Action] 8장. 컬렉션 API 개선"
excerpt: "Java 8과 9에 추가된 컬렉션 API를 통해 관련 코드들을 더욱 간결하게 짤 수 있다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-03
last_modified_at: 2021-03-03
---

## 1. Collection 팩토리

> ListFactory.java

```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
List<String> friends2 = List.of("Raphael", "Olivia", "Thibaut");
```

* ``asList()``는 고정 크기의 불변 리스트를 반환하기 때문에 해당 리스트에 원소를 추가하거나 삭제할 수 없다.
  * Java 9에서 추가된 ``of()`` 또한 동일한 제약을 가지며, null 원소를 불허한다.

> SetFactory.java

```java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
Set<String> friends2 = Stream.of("Raphael", "Olivia", "Thibaut")
           .collect(Collectors.toSet);
Set<String> friends3 = Set.of("Raphael", "Olivia", "Thibaut");
```

* Java 9의 ``Set.of()``를 통해 생성한 Set 또한 불변이며, 중복되는 원소들로 생성을 시도하는 경우 예외가 발생한다.
  * 다른 방식으로 생성하는 경우 Set은 가변이다.

> MapFactory.java

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
Map<String, Integer> ageOfFriends2 = Map.ofEntries(entry("Raphael", 30),
                       entry("Olivia", 25),
                       entry("Thibaut", 26));
```

* 마찬가지로 Java 9에 추가된 메서드로서 불변 Map을 생성한다.
* ``of()`` 등을 정의한 컬렉션 인터페이스들은 가변 인자를 받는 메서드 외에도 원소를 1개 ~ n개만 고정으로 받을 수 있는 여러 메서드들을 오버로딩해두었다.
  * 가변 인자를 사용하면 내부적으로 가변 배열을 할당해 사용하는 등 여러 오버헤드가 발생한다.
  * 자주 사용하는 범위에 한해서 고정 인자를 받는 메서드를 정의해둠으로써 오버헤드를 줄인다.

<br>

## 2. Lis와 Set 처리

> MutableExample.java

```java
for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}
```

* 위 코드는 멀티 스레드 환경에서 ConcurrentModification이 발생할 수 있다.

> MutableExample.java

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction transaction = iterator.next();
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}
```

* 내부적으로 컬렉션을 순회하는 Iterator 객체와 삭제 메서드를 호출하는 Collection 객체가 각각 컬렉션을 관리한다.
* Iterator와 컬렉션 상태의 동기화가 쉽게 깨진다.
  * Transaction 객체가 아닌 Iterator 객체가 직접 ``remove()``를 호출하면 해결할 수 있으나 코드의 가독성이 좋지 못하다.

> ListFactory.java

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

* Java 8에 추가된 메서드들을 통해 리스트 원소를 삭제하거나 교체할 때의 코드를 가독성 좋게 표현하면서 동시에 동시성 문제를 해결할 수 있다.

<br>

## 3. Map 처리

> MapFactory.java

```java
for (Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}

ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

* Map을 순회할 때에도 ``forEach()``를 사용하면 코드를 간결화할 수 있다.

> MapFactory.java

```java
favouriteMovies.entrySet()
    .stream()
    .sorted(Entry.comparingByKey())
    .forEachOrdered(System.out::println);
```

* ``Entry.comparingByKey()`` 및 ``Entry.comparingByValue()``를 통해 Map의 엔트리를 쉽게 정렬할 수 있다.
* HashMap은 키의 해시코드에 따라 엔트리 저장 공간에 접근하는데, 이러한 저장 공간을 전형적으로 LinkedList를 사용했다.
  * 여러 키들이 같은 해시코드 값을 반환하는 경우 Map의 성능이 떨어진다.
  * Java 8은 이러한 저장 공간이 너무 커지는 경우 정렬 트리로 대체하여 성능을 향상시킨다.
    * Map의 모든 키들이 Comparable 인터페이스를 구현한 경우에만 사용 가능하다.

> MapFactory.java

```java
System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
```

* Map에서 특정 Key의 값이 없는 경우 기본 값을 반환한다.

> MapFactory.java

```java
lines.forEach(line -> dataToHash.computeIfAbsent(line, this::calculateDigest));
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars");
```

* Map에 특정 키가 없는 경우 초기화 람다를 수행하고 생성된 값을 Map에 저장한 다음 반환한다.
* 키가 존재하는 경우 해당 키에 부합하는 값을 정상적으로 ``get()``처럼 반환한다.
* 비슷하게 ``computeIfPresent()`` 등이 있다.
  * 현재 키에 부합하는 값이 null이 아니면 람다를 수행하는데, 람다 Function의 수행 결과가 null이면 해당 엔트리 맵핑이 Map에서 삭제된다.

> MapFactory.java

```java
favouriteMovies.remove(key, value);
```

* Map이 특정 키를 포함하는지 그리고 해당 키에 부합하는 값이 null이 아닌지 등을 체크하고 키를 삭제하면 코드가 장황해진다.
* ``remove()`` 메서드를 통해 간략하게 엔트리를 삭제한다.

> MapFactory.java

```java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

* 모든 엔트리의 값을 주어진 BiFunction을 적용한 값으로 변경한다.
* 비슷한 ``replace()`` 메서드는 Map에 주어진 키가 존재하는 경우 값을 변경한다.

> MapFactory.java

```java
Map<String, String> family = Map.ofEntries(entry("Teo", "Star Wars"));
Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```

* 두 개의 Map을 합칠 때 ``putAll()`` 메서드를 사용할 수 있으나, 중복되는 키가 있어서는 안 된다.

> MapFactory.java

```java
Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```

* ``merge()``와 ``forEach()``를 통해서도 두 개의 Map을 병합할 수 있다.
* 키가 중복되는 경우 중복을 해결하는 방법을 람다로 제공할 수 있는 등 조금 더 유연한 컨트롤이 가능하다.
  * 위 예제 코드의 수행 결과로 Cristina 키의 값은 James Bond & Matrix가 된다.

> MapFactory.java

```java
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if (count == null) {
   moviesToCount.put(movieName, 1);
}
else {
   moviesToCount.put(moviename, count + 1);
}

//복잡한 Map 초기화

moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```

* Map을 초기화하는 코드를 간략화할 수 있다.
  * movieName에 해당하는 키가 없다면 기본 값인 1L을 Map에 저장하고, 중복이 있다면 카운트를 1씩 증가시킨다.

<br>

## 4. ConcurrentHashMap

ConcurrentHashMap은 멀티 스레드 환경에서 스레드 안전하게 Map의 추가 및 수정 등의 연산을 할 수 있도록 지원하는 클래스다. 이러한 연산을 수행할 때, 내부 자료 구조에서 특정한 부분만 락을 걸기 때문에 기존의 SynchronizedCollection에서 제공하는 HashMap보다 성능이 뛰어나다.

> ConcurrentHashMapTest.java

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

* ``forEachXXX()``, ``reduceXXX()``, ``searchXXX()`` 메서드를 지원한다.
  * XXX에 해당하는 것은 Key, Value, Entry다.
  * XXX가 생략된 경우 메서드 연산 수행시 Key와 Value를 모두 참조한다.
* 해당 연산들은 ConcurrentHashMap 상태에 락을 걸지 않는다.
  * 해당 연산들에게 주어지는 Function 함수형 인터페이스는 연산 도중 변경될 수 있는 특정 순서나 값 및 객체 등에 의존해서는 안 된다.
* 해당 연산들에 대해 병렬성 문턱값(Threshold)를 명시해야 한다.
  * Map의 크기가 해당 문턱값보다 작은 경우 연산은 순차적으로 수행된다.
  * 문턱값이 1이면 공통 스레드 풀을 이용하여 병렬화를 최대한 이용한다.
  * 문턱값이 ``Long.MAX_VALUE``라면 싱글 스레드에서 연산이 수행된다.
    * 일반적으로 위 두 값을 많이 사용한다.
* 위 예제는 Map의 값들 중에서 최대 값을 찾는 코드이다.
* 오토박싱을 사용하지 않는 기본형 특화 ``reduce()`` 연산들 또한 존재한다.
  * ``reduceValuesToInt()``, ``reduceKeysToLong()`` 등.
* ``mappingCount()``를 지원한다.
  * ``size()``와 다르게 Map의 맵핑 엔트리의 개수를 long 타입으로 반환한다.
* ``keySet()`` 메서드를 통해 Map의 View를 반환한다.
  * 반환된 Set은 실제 Map에서 사용하고 있는(레퍼런스가 같은) Set이기 때문에, 어느 한쪽에서 Set을 변경하면 다른 한 쪽에도 똑같이 영향이 간다.
  * ``newKeySet()``을 통해 Set을 만들 수 있다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
