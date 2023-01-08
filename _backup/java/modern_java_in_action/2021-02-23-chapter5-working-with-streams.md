---
title: "[Modern Java in Action] 5장. 스트림 활용"
excerpt: "스트림 API가 제공하는 인터페이스를 통해 데이터를 필터링, 슬라이싱, 맵핑해보자."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-02-23
last_modified_at: 2021-02-23
---

## 1. 필터링

``filter()`` 메서드는 인자로 받는 Predicate에 부합하는 원소들만을 걸러준다. ``distinct()``는 ``hashCode()`` 및 ``equals()`` 메서드를 기반으로 스트림들 중 고유한 원소들만을 걸러준다.

<br>

## 2. 슬라이싱

> Dishes.java

```java
List<Dish> specialMenu = Arrays.asList(
      new Dish("season fruit", true, 120, Dish.Type.OTHER),
      new Dish("prawns", false, 300, Dish.Type.FISH),
      new Dish("rice", true, 350, Dish.Type.OTHER),
      new Dish("chicken", false, 400, Dish.Type.MEAT),
      new Dish("french fries", true, 530, Dish.Type.OTHER));
```

* ``filter()``를 사용하면 스트림 전체를 순회하면서 해당 Predicate가 모든 원소에게 적용된다.
  * 스트림의 크기가 너무 큰 경우 비효율적일 수도 있다.
* 예시처럼 원소가 정렬되어 있는 경우 Java 9이 제공하는 ``takeWhile()`` 및 ``dropWhile()`` 메서드를 통해 필터링 성능을 높일 수 있다.

> Dishes.java

```java
List<Dish> slicedMenu1 = specialMenu.stream()
      .takeWhile(dish -> dish.getCalories() < 320)
      .collect(toList());

List<Dish> slicedMenu2 = specialMenu.stream()
      .dropWhile(dish -> dish.getCalories() < 320)
      .collect(toList());
```

* ``takeWhile()``은 Predicate에 부합하는 원소들만을 걸러서 슬라이싱해주지만, 해당 Predicate가 false를 반환하는 원소를 만나면 멈춘다.
* ``dropWhile()``은 Predicate가 false를 반환하는 원소들은 버리다가, true를 반환하는 원소를 만나면 그 즉시 모든 남아있는 스트림들을 반환한다.
* 스트림의 원소들이 정렬되었음을 확신할 수 있는 경우 사용하기 좋다.

### 2.1. 기타

* ``limit()``으로 스트림 원소들 중 일부만을 가져올 수 있다.
  * 정렬된 경우 가장 선두의 원소들이, 정렬되지 않은 경우 순서가 보장되지 않은 원소들이 리턴된다.
* ``skip()``은 선두 n개의 원소들을 생략한 스트림을 반환한다.
  * 스트림 크기가 매개변수 n보다 적은 경우 빈 스트림이 반환된다.

<br>

## 3. 맵핑

> Mapping.java

```java
List<String> dishNames = menu.stream()
      .map(Dish::getName)
      .collect(toList());
```

* ``map()`` 메서드는 인자로 주어진 Function을 각 원소들에 적용하며, 각 원소들을 새로운 원소로 맵핑한다.
* Transforming은 수정보다는 새로운 버전을 만든다의 뉘앙스가 강하기 때문에 **Mapping**이란 단어를 사용했다.

### 3.1. Stream 평탄화

> Mapping.java

```java
words.stream()
      .map(word -> word.split("")) //Stream<String[]>
      .map(Arrays::stream) //Stream<Stream<String>>
      .distinct();
```

* List\<String\>에 담긴 문자열 단어들을 잘라 한 문자씩 List에 담으려고 한다.
* 결과는 Stream\<String\>이 아닌, Stream\<Stream\<String\>\>이 된다.

> Mapping.java

```java
words.stream()
      .map(word -> word.split("")) //Stream<String[]>
      .flatMap(Arrays::stream) //Stream<String>
      .distinct();
```

* ``flatMap()``은 스트림의 원소를 다른 스트림으로 변경하고, 이렇게 생성된 스트림들을 연결하여 하나의 스트림으로 평탄화해준다.

<br>

## 4. 탐색 및 일치

``allMatch()``, ``anyMatch()``, ``noneMatch()``, ``findFirst()``, ``findAny()`` 등은 종단 연산이다. 인자로 받은 Predicate를 통해 원소 혹은 일치 여부를 반환한다. 해당 연산들은 숏 서킷이 적용될 수 있다.

* 숏 서킷(Short-Circuiting)이란?
  * 여러 논리 연산자들이 체이닝되어 있을 때, 앞의 수식 결과가 전체 수식의 결과를 확정 짓게 되면 뒤의 수식은 검사하지 않는 것이다.
  * ``a && b && c``일 때, a가 false이면 b와 c는 검사하지 않아도 된다.
* 숏 서킷이 적용되는 연산자들은 결과를 생산하기 위해 스트림 **전체**를 처리할 필요는 없다.
  * ``limit()`` 등도 숏 서킷이 적용되는 연산자이다.

### 4.1. Optional

Optional은 값이 존재하는지 부재하는지를 표현하기 위한 컨테이너 클래스이다. 값이 없을 때 null을 반환하는 코드들은 NPE를 유발하기 쉽기 때문에 Optional로 감싸 사용한다. 탐색 관련 스트림 메서드들은 원하는 찾고자 하는 원소가 존재하지 않을 수 있기 때문에 Optional을 반환한다.

* 단, ``findFirst()``는 병렬화시 제약 사항이 많기 때문에 어떤 원소가 반환되도 상관없는 경우 병렬화 제약 사항이 덜한 ``findAny()``를 사용하자.

<br>

## 5. 리듀싱

리듀싱 연산은 스트림 원소들을 반복적으로 결합하여 마침내 단일 원소를 생성한다. 스트림이 하나의 원소로 축소(Reduce)됨을 의미한다. 외부 순회를 통한 리듀싱보다 병렬화 등에서 장점이 있으나 제약 사항이 존재한다.

* 인자로 주어진 람다식은 변수 등의 상태를 변경할 수 없다.
* 연산은 어떤 순서로도 실행될 수 있도록 교환 법칙이 성립해야 한다.

> Reducing.java

```java
List<Integer> numbers = Arrays.asList(3, 4, 5, 1, 2);
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int sum2 = numbers.stream().reduce(0, Integer::parseInt);
```

* 첫 번째 인자는 초기값이며, 두 번째 인자는 BinaryOperator이다.
* BinaryOperator는 두 원소들을 합쳐 새로운 원소를 만들어내는 작업을 반복한다.
  * 0 + 3, 3 + 4, 7 + 5, 12 + 1, 13 + 2 순으로 15를 생성한다.
  * 따라서 곱셈을 적용할 때는 초기값을 1로 설정하라.

> Reducing.java

```java
List<Integer> numbers = Arrays.asList(3, 4, 5, 1, 2);
Optional<int> sum = numbers.stream().reduce((a, b) -> a + b);
```

* 초기값을 받지 않은 오버로딩된 메서드는 Optional을 반환한다.
  * 스트림이 비어있는 경우 리듀싱 연산은 초기값이 없어 합계를 반환할 수 없기 때문이다.

> Comparing.java

```java
int max = numbers.stream().reduce(0, (a, b) -> Integer.max(a, b));
System.out.println(max);

Optional<Integer> min = numbers.stream().reduce(Integer::min);
min.ifPresent(System.out::println);
```

* 리듀싱 연산은 새로운 값을 스트림의 다음 원소와 사용함으로써 새로운 값을 생성한다.
  * 전체 스트림을 소비할 때 까지 대소 비교를 진행하면 최대값이나 최소값을 찾을 수 있다.

### 5.1. Stateful vs Stateless

``map()``이나 ``filter()``같은 연산은 입력 스트림의 원소들을 취해 0개 혹은 1개의 결과를 출력 스트림에 생산한다. 대게 Stateless하다. 그러나 리듀싱 연산들은 결과를 축적하기 위해 내부 상태를 가지게 된다.

``sorted()`` 및 ``distinct()``는 작업을 수행하기 전에 스트림을 복사한다. 출력 스트림에 원소를 추가하기 전에 모든 원소들이 버퍼링되어야 하며, 저장 요구사항은 비한정적이다. 스트림이 크거나 무한대인 경우 문제가 될 수 있으며, 이러한 연산을 Stateful하다고 한다.

<br>

## 6. Numeric Stream

> NumericStream.java

```java
int calories = menu.stream()
      .map(Dish::getCalories)
      .reduce(0, Integer::sum);
```

* Integer는 합계 연산을 진행하기 전에 기본형으로 언박싱되는 비용을 수반하게 된다.
* Stream 인터페이스는 ``sum()``과 같은 합계 등의 메서드를 정의하지 않았다.
  * Stream\<T\>를 사용하는데 Stream\<Dish\>는 합계를 구할 수 없지 않은가.
* Stream API는 숫자와 관련된 스트림을 작업할 때 유용한 기본 자료형 특화 API를 제공한다.

### 6.1. 기본 자료형 스트림

IntStream, LongStream, DoubleStream 등을 제공한다. Integer 등 래핑 클래스 원소를 사용하는 일반 Stream과 다르게 해당 스트림들은 기본 자료형을 스트림의 원소로 사용한다. 이로 인해 오토박싱으로 인한 비용을 절감할 수 있다.

또한 위의 기본 자료형 특화 스트림 인터페이스들은 ``sum()``, ``max()`` 등 자주 활용하는 정수 스트림 연산을 제공한다.

* 일반 스트림을 기본 자료형 스트림으로 변경하는 경우 ``mapToXXX()`` 메서드를 사용한다.
* ``boxed()``나 ``mapToObj()``를 사용하면 기본 자료형 스트림을 일반 스트림으로 박싱할 수 있다.
  * ``boxed()``는 단순히 기본 자료형 스트림을 매칭되는 래핑 클래스 스트림으로 변환하지만, ``mapToObj()``는 원하는 타입의 스트림으로 변경 가능하다.

> NumericStream.java

```java
OptionalInt maxCalories = menu.stream()
      .mapToInt(Dish::getCalories)
      .max();
```

Optional 또한 기뵨 자료형에 특화된 OptionalInt, OptionalDouble, OptionalLong 등을 제공한다.

> NumericStream.java

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
    .filter(n -> n % 2 == 0);
```

* 특정 범위의 정수 스트림을 생성할 때 ``range()`` 혹은 ``rangeClosed()``를 사용한다.
  * 두 메서드의 차이는 두 번째 파라미터 범위값 포함 유무이다.

<br>

## 7. 스트림 생성

> BuildingStreams.java

```java
Stream<String> stream = Stream.of("Java 8", "Lambdas", "In", "Action");
Stream<String> emptyStream = Stream.empty();

int[] numbers = { 2, 3, 5, 7, 11, 13 };
System.out.println(Arrays.stream(numbers).sum());

//Java 8의 단점
String value = System.getProperty("home");
Stream<String> homeValueStream1 = value == null ? Stream.empty() : Stream.of(value);

//Java 9부터 지원되는 nullable
Stream<String> homeValueStream2 = Stream.ofNullable(value);
```

* ``Stream.empty()``는 null에 대해 안전하다.
  * Stream에 null이 포함된 경우 참조 및 연산 도중 NPE가 발생할 수 있기 때문에 이용한다.
* Java 9 부터는 스트림을 생성할 때 null이 포함되어 있어도 안전하게 처리해주는 ``ofNullable()``을 지원한다.

### 7.1. I/O 활용

> BuildingStreams.java

```java
try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
      long uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                              .distinct()
                              .count();
} catch (IOException e) {
}
```

* ``Files.lines()``는 주어진 파일의 각 라인들이 원소가 되는 스트림을 반환한다.
* Stream 인터페이스는 AutoCloseable을 확장했기 때문에 try-with-resources 구문을 사용할 수 있다.

### 7.2. 무한 스트림

``Stream.iterate()`` 및 ``Stream.generate()``로 스트림을 생성할 수 있으며, 개수 제한을 걸지 않으면 무한 스트림이 생성된다. 스트림 전체를 다뤄야 하는 ``sorted()``나 ``reduce()`` 등의 연산은 무한 스트림에서 사용할 수 없다.

> BuildingStreams.java

```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

* ``iterate()``는 초기값과 람다식을 받으며, 첫 번째 스트림 원소는 초기값으로 명시된 0이 된다.
  * 이후 ``limit()``에 명시된 개수까지 람다식으로 리듀싱하며 스트림 원소들을 생성해나간다.
  * 결과가 이전 어플리케이션에 의존하기 때문에 해당 연산은 근본적으로 순차적이다.
* 무한 스트림을 생성할 수 있는 이유는 스트림의 값들은 종단 연산이 호출되는 시점에 필요에 따라 on-demand로 계산되며, 심지어는 평생 계산될 수 있기 때문이다.
* 연속된 날짜와 같이 연속된 값들의 시퀀스를 생산하는 경우 ``iterate()``를 사용한다.

> BuildingStreams.java

```java
IntStream.iterate(0, n -> n < 100, n -> n + 2)
      .forEach(System.out::println);
```

* Java 9 부터는 두 번째 인자로 받은 Predicate를 통해 스트림이 생성되는 반복 조건을 명시할 수 있다.
  * n이 100 미만일 때 까지 스트림을 생성한다.

> BuildingStreams.java

```java
IntStream.iterate(0, n -> n + 2)
      .filter(n -> n < 100)
      .forEach(System.out::println);
```

* ``filter()``는 숫자가 계속 증가한다는 사실을 모르기 때문에 위 코드는 무한 스트림이 되버린다.

> BuildingStreams.java

```java
IntStream.iterate(0, n -> n + 2)
    .takeWhile(n -> n < 100)
    .forEach(System.out::println);
```

* ``takeWhile()`` 등 Java 9이 제공하는 숏 서킷이 적용될 수 있는 슬라이싱 메서드를 사용하면 유한 스트림이 된다.

> BuildingStreams.java

```java
Stream.generate(Math::random)
      .limit(10)
      .forEach(System.out::println);
```

* ``generate()``는 Supplier를 인자로 받는다.
* 해당 Supplier는 Stateless일 필요는 없으나 Stateful하면 병렬화시 안전하지 못하다.

> Fibonacci.java

```java
IntSupplier fib = new IntSupplier() {
     private int previous = 0;
     private int current = 1;

     @Override
     public int getAsInt() {
       int nextValue = previous + current;
       previous = current;
       current = nextValue;
       return previous;
     }
};
```

* 피보나치 수열에 대한 Supplier를 익명 클래스로 작성할 때, 인스턴스는 상태를 가지는 가변이 될 수 있다.

> Fibonacci.java

```java
Stream.iterate(new int[] { 0, 1 }, t -> new int[] { t[1], t[0] + t[1] })
    .limit(10)
    .map(t -> t[0])
    .forEach(System.out::println);
```

* 람다식은 이러한 부작용이 없다.
* ``iterate()`` 메서드는 완벽한 불변으로서 매 순회마다 새로운 객체를 생성한다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
