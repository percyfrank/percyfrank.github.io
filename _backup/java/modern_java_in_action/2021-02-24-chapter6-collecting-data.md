---
title: "[Modern Java in Action] 6장. 스트림으로 데이터 수집"
excerpt: "Collector 인터페이스를 사용하여 데이터를 간단하게 수집해본다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-02-24
last_modified_at: 2021-02-24
---

## 1. Stream API의 collect

Stream API의 ``collect()`` 메서드는 Collector 인터페이스를 인자로 받아 스트림 원소들을 수집(그룹핑 및 파티셔닝 등)한다. 외부 반복문 순회(명령형)를 통한 컬렉션 데이터 수집에 비해, Stream API를 통한 데이터 수집은 코드를 함수형으로 표현할 수 있어 코드가 간결하고 가독성이 우수해진다. 데이터 수집을 위한 구현 방식(How)보다는 얻고자 하는 결과 서술(What)에 중점을 두고 코드를 작성하기 때문이다.

Collector 인터페이스는 ``collect()``메서드가 데이터를 수집할 때 사용하는 기준이다. ``collect()`` 메서드를 호출하면 내부적으로 리듀싱 연산이 발생한다. Collectors 유틸 클래스는 빈번하게 사용되는 Collector 구현체를 정적 팩토리 메서드를 통해 제공하며, 원한다면 사용자가 직접 구현한 Collector 인터페이스를 사용해도 된다.

``collect()``의 기능들은 Stream API가 제공하는 기능들과 일부 중복되는 부분이 있으며 심지어 Stream API의 기능들이 더 직관적인 경우가 많다. 그러나 Collector 인터페이스는 더 높은 수준의 추상화 및 일반화를 제공하며, 코드를 커스터마이징하고 재사용할 수 있는 장점이 있다.

<br>

## 2. 리듀싱 및 요약

> Dishes.java

```java
long howManyDishes = menu.stream().collect(Collectors.counting());
long howManyDishes = menu.stream().count(); //동일 효과
```

* ``collect()`` 종단 연산은 컬렉션뿐만 아니라 단일 값도 반환할 수 있다.

> Dishes.java

```java
Comparator<Dish> comparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> maxCalorieDish = menu.stream().collect(Collectors.maxBy(comparator));
Optional<Dish> minCalorieDish = menu.stream().collect(Collectors.minBy(comparator));
```

* Comparator를 받는 ``maxBy()`` 및 ``minBy()``를 통해 최대 혹은 최소값을 찾을 수 있다.

> Dishes.java

```java
int totalCalories = menu.stream().collect(Collectors.summingInt(Dish::getCalories));
double avgCalories = menu.stream().collect(Collectors.averagingInt(Dish::getCalories));
IntSummaryStatistics menuStatistics = menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
```

* ``summingXXX()``는 Function을 받아 XXX에 해당하는 자료형 값들의 합계를 반환한다.
* ``averagingXXX()``는 Function을 받아 XXX에 해당하는 자료형 값들의 평균을 반환한다.
* ``summarizingXXX()``는 Function을 받아 XXX에 해당하는 자료형 값들의 요약 정보 객체인 XXXSummaryStatistics를 반환한다.
  * getter를 통해 최대, 최소, 합계, 평균, 개수 등 여러 요약 정보에 접근할 수 있다.

> Dishes.java

```java
String shortMenu = menu.stream().collect(Collectors.joining());
String shortMenu2 = menu.stream().map(Dish::getName).collect(Collectors.joining(", "));
```

* ``joining()`` 메서드는 스트림 객체들의 ``toString()``을 통해 반환되는 문자열들을 연결해 하나의 문자열로 만들어준다.
  * 오버로딩된 메서드를 통해 접두어, 접미어, 구분자 등을 추가할 수 있다.
* 내부적으로 StringBuilder를 사용한다.

> Dishes.java

```java
int totalCalories = menu.stream()
        .collect(Collectors.reducing(0, Dish::getCalories, Integer::sum));

Optional<Dish> mostCalorieDish = menu.stream()
        .collect(Collectors.reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

* Stream API의 ``reduce()``와 유사하게 동작하는 ``reducing()``을 사용할 수 있다.
  * 인자로 초기값과 객체를 변환시킬 Function 및 리듀싱 연산에 사용될 BinaryOperator Accumulator를 입력받는다.
* 초기값 없이 Accumulator만을 받는 오버로딩 메서드는 스트림의 가장 처음 아이템을 시작점으로 지정하여 리듀싱 작업을 진행한다.
  * Stream API의 오버로딩된 ``reduce()`` 처럼 스트림이 비어있는 경우를 감안해 Optional 객체를 반환한다.

> Collectors.java

```java
public static <T> Collector<T, ?, Long> counting() {
    return reducing(0L, e -> 1L, Long::sum);
}
```

* Collectors가 제공하는 ``counting()`` 정적 팩토리 메서드 또한 내부적으로는 ``reducing()``을 사용한다.

### 2.1. reduce() vs collect()

Stream API의 ``reduce()``는 두 값을 결합해 새로운 값 하나를 생성하는 불변 리듀싱이다. 반면 ``collect()``는 결과를 누계하기 위해 컨테이너를 가변시키도록 고안되었다.

> AntiPattern.java

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(
         new ArrayList<Integer>(),
         (List<Integer> l, Integer e) -> {
                 l.add(e);
                 return l; },
         (List<Integer> l1, List<Integer> l2) -> {
                 l1.addAll(l2);
                 return l1; });
```

* 위 예제는 Accumulator로 사용되는 List 컨테이너를 가변시키는 잘못된 ``reduce()`` 사용이다.
* ``collect()`` 작업을 ``reduce()``로 대체할 수 있을지라도 병렬화 작업에서는 제대로 작동하지 않는다.
  * 여러 스레드가 리스트에 대해 ConcurrentModification을 수행할 때 List는 스레드 안전하지 못하고 오염될 수 있다.
* 가변 컨테이너를 통해 리듀싱 작업을 할 때 ``collect()``가 더 병렬 처리에 친화적이다.

<br>

## 3. 그룹핑

> Dishes.java

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType));
```

* ``groupingBy()`` 메서드에 제공한 classifier Function이 반환하는 결과값 Key들을 기준으로 스트림을 여러 그룹으로 분리한다.
* 내부적으로는 ``groupingBy(classfier, toList())``를 호출한다.

> Dishes.java

```java
Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
        .filter(dish -> dish.getCalories() > 500)
        .collect(Collectors.groupingBy(Dish::getType));
//{OTHER=[french fries, pizza], MEAT=[pork, beef]}

Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
        Collectors.filtering(dish -> dish.getCalories() > 500, Collectors.toList())));
//{OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}

Map<Dish.Type, List<String>> dishNamesByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
        Collectors.mapping(Dish::getName, Collectors.toList())));

Map<String, List<String>> dishTags = new HashMap<>();
Map<Dish.Type, Set<String>> dishNamesByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
        Collectors.flatMapping(dish -> dishTags.get(dish.getName()).stream(), Collectors.toSet())));
```

* ``filter()``를 사용하여 그룹핑했을 때 Fish Type의 음식들은 해당 Predicate를 충족하지 못한다.
  * 결과로 반환된 Map에서는 Fish와 관련된 엔트리가 누락된다.
* Java 9에 도입된 ``filtering()``은 엔트리 누락없이 필터링을 적용하여 그룹핑해준다.
* ``mapping()``과 ``flatMapping()``은 객체를 맵핑하여 그룹핑해준다.
* 값으로 사용할 컬렉션의 구현체를 지정해주고 싶다면 ``Collectors.toCollection(HashSet::new)`` 등을 사용한다.

### 3.1. 멀티레벨 그룹핑

``groupingBy()``의 오버로딩 메서드들은 Function 이외에도 mapFactory 및 downstream Collector 등을 인자로 받을 수 있다.

> Dishes.java

```java
Map<Dish.Type, Long> typesCount = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType, Collectors.counting()));

Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
        Collectors.maxBy(Comparator.comparingInt(Dish::getCalories))));

Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
        .collect(Collectors.groupingBy(Dish::getType,
        Collectors.collectingAndThen(
        Collectors.maxBy(Comparator.comparingInt(Dish::getCalories)),
        Optional::get)));
```

* ``maxBy()`` 특성으로 인해 Optional이 반환되지만, 특정 타입에 대한 음식이 없는 경우 ``Optional.empty()``를 Map의 값으로 가지지 않는다.
  * 해당 타입 Key에 대한 엔트리는 맵에 수집될 때 누락되버리기 때문에 Optional이 의미가 없다.
  * 따라서 바로 ``get()``으로 아이템을 꺼내도 안전하다.
* ``collectingAndThen()``은 downstream Collector로 리듀싱한 그룹핑 결과를 finisher Function을 통해 변환한다.

<br>

## 4. 파티셔닝

> Dishes.java

```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream()
        .collect(Collectors.partitioningBy(Dish::isVegetarian));

Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
        .collect(Collectors.partitioningBy(Dish::isVegetarian,
        Collectors.groupingBy(Dish::getType)));

Map<Boolean, Dish> mostCaloricPartitionedByVegetarian = menu.stream()
        .collect(Collectors.partitioningBy(Dish::isVegetarian,
        Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparingInt(Dish::getCalories)),
        Optional::get)));
```

* 파티셔닝은 Predicate를 기준으로 스트림을 그룹핑하는 기능이다.
* ``partitioningBy()``의 오버로딩 메서드는 downstream Collector도 인자로 받기 때문에 추가적인 그룹핑 작업을 수행할 수 있다.

<br>

## 5. Collector 인터페이스

Collector 인터페이스를 직접 구현하면 리듀싱 연산을 커스터마이징할 수 있다.

> Collector.java

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

* T는 스트림 원소의 지네릭스 타입이다.
* A는 Accumulator의 타입으로 컬렉션 연산을 누계하는 객체이다.
* R은 컬렉션 연산의 결과로 반환되는 결과 객체 타입이다.

> ToListCollector.java

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {

    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (list1, list2) -> {
            list1.addAll(list2);
            return list1;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));
    }
}
```

* ``supplier()``는 비어있는 Accumulator를 반환하는 메서드다.
* ``accumulator()``는 리듀싱 연산을 수행하는 함수를 반환하는 메서드다.
* ``finisher()``는 누계 처리의 마지막에 호출되는 함수를 반환하며, Accumulator 객체를 결과인 R 객체로 변환하는 역할이다.
  * 위의 경우 A와 R이 같기 때문에 ``Function.identity()``를 사용했다.
* ``characteristics()``은 Collector의 행동을 정의한다.
  * 스트림의 리듀싱 연산이 병렬로 처리 될 수 있는지, 혹은 추가적인 최적화가 가능한지 등을 EnumSet으로 명시한다.
  * 키워드로는 UNORDERED, CONCURRENT, IDENTITY_FINISH가 있다.
    * CONCURRENT 플래그를 사용해도 UNORDERED 플래그를 사용하지 않는다면 순서가 없는 데이터 소스에서만 병렬 리듀싱 연산이 가능하다.
    * IDENTITY_FINISH 플래그는 A와 R이 같기 때문에 비검사 형변환을 해도 괜찮다는 등의 추가적인 최적화 힌트를 제공한다.

![image](https://user-images.githubusercontent.com/56240505/109419917-2e9de880-7a13-11eb-93c1-b8f7736994fd.png)
![image](https://user-images.githubusercontent.com/56240505/109419933-46756c80-7a13-11eb-9da0-6c3746286226.png)

* ``combiner()``는 스트림이 서브 파트로 분할되어 병렬로 처리될 경우 서브 파트들의 Accumulator를 어떻게 병합할지 정의한 메서드다.
  * 스트림의 병렬 리듀싱 연산을 가능하게 해주는 메서드로서, fork-join 프레임워크와 Spliterator 추상화를 사용한다.
* 원본 스트림은 특정 조건에 부합될 때 까지 재귀적으로 분할된다.
  * 서브 파트가 너무 적거나 코어보다 너무 많게 분할되면 성능이 떨어진다.

> Main.java

```java
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());

//혹은

List<Dish> dishes = menuStream.collect(ArrayList::new, List::add, List::addAll);
```

* 구현한 Collector 인터페이스를 인자로 제공하여 데이터를 수집할 수 있다.
* supplier와 accumulator 및 combiner를 입력받는 ``collect()`` 오버로딩 메서드를 통해 동일한 기능을 구현할 수 있다.
  * 다소 가독성이 떨어지며, ``characteristics()``에 해당하는 기능은 사용할 수 없다.
  * 반면 Collector 인터페이스 구현을 클래스로 관리하면 재사용이 가능하며 코드 중복을 피할 수 있다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
