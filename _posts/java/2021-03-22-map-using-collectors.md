---
title: "Stream API : groupingBy로 Map 수집하기"
excerpt: "강력한 함수형 데이터 처리 기능을 사용해보자."
categories:
  - Java
tags:
  - Java
date: 2021-03-22
last_modified_at: 2021-03-22
---

## 1. groupingBy(classifier)

> Collectors.java

```java
static <T,K> Collector<T,?,Map<K,List<T>>> groupingBy(Function<? super T,? extends K> classifier)
```

T를 K로 맵핑하고, K에 저장된 List에 T를 저장한 Map을 생성한다.

* **classfier** : groupingBy를 위한 기준 값을 가져오는 function이다.

> Test.java

```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 2, 2, 3, 3, 4, 5, 6, 6);
    Map<Integer, List<Integer>> map = numbers
            .stream()
            .collect(Collectors.groupingBy(n -> n));
    System.out.println(map);
    //{1=[1], 2=[2, 2, 2], 3=[3, 3], 4=[4], 5=[5], 6=[6, 6]}
}
```

위의 예제는 리스트의 각 숫자들(T)을 숫자 그 자체(K)로 그룹핑한 다음, 같은 그룹(K)에 속한 리스트(T)를 생성한다. 숫자를 Key로, 리스트를 Value로 갖는 Map을 생성한다.

> Test.java

```java
public static void main(String[] args) {
    List<Person> persons = Arrays.asList(new Person("a", "male"), new Person("b", "male")
            , new Person("c", "male"), new Person("d", "female"), new Person("e", "female"));
    Map<String, List<Person>> personBySex = persons.stream()
            .collect(Collectors.groupingBy(Person::getSex));
    personBySex.forEach((key, value) -> {
        System.out.print(key + " ");
        value.forEach(v -> System.out.print(v.getName() + " "));
        System.out.println();
        /*
        female d e
        male a b c
        */
    });
}
```

위의 예제는 리스트의 각 학생들(T)을 성별(K)로 그룹핑한 다음, 같은 그룹(K)에 속한 학생 리스트(T)를 생성한다. 성별을 Key로, 리스트를 Value로 갖는 Map을 생성한다.

<br>

## 2. groupingBy(classifier, mapFactory, downstream)

> Collectors.java

```java
static <T,K,D,A,M extends Map<K,D>> Collector<T,?,M> groupingBy(Function<? super T,? extends K> classifier, Supplier<M> mapFactory,
    Collector<? super T,A,D> downstream)
```

T를 K로 맵핑하고, Supplier가 제공하는 Map에서 K키에 저장된 D객체에 T를 누적한다.

* **mapFactory** : 함수의 수행 결과로서 새롭게 만들어지는 Map을 생성하는 function이다.
* **downstream** : groupingBy의 결과로서 얻어지는 결과 Collector이다.

> Test2.java

```java
public static void main(String[] args) {
    String[] persons = {"a", "b", "a", "b", "b", "b"};
    Map<String, Long> countsByPerson = Arrays.stream(persons)
            .collect(Collectors.groupingBy(person -> person, HashMap::new, Collectors.counting()));
    System.out.println(countsByPerson);
    //{a=2, b=4}
}
```

배열의 각 문자열(T)를 문자열 그 자체(K)로 맵핑하고, HashMap의 문자열 Key(K)에 저장된 Long 객체(D)에 ``counting()`` (T, reduce 메서드)을 누적한다.

> Test2.java

```java
public static void main(String[] args) {
    String[] persons = {"a", "b", "a", "b", "b", "b"};
    Map<String, List<String>> countsByPerson = Arrays.stream(persons)
            .collect(Collectors.groupingBy(person -> person, HashMap::new, Collectors.toList()));
    System.out.println(countsByPerson);
    //{a=[a, a], b=[b, b, b, b]}
}
```

downstream을 counting에서 list로 수정하면 위의 결과를 얻을 수 있다.

<br>

## 3. 그 외 Map의 유용한 메서드

> Test3. java

```java
map.forEach((key, value) -> {
    System.out.println(key + " " + value);
});

map.computeIfAbsent((key) -> {
    List<String> list = new ArrayList<>();
    list.add("default value");
    if (...)
      ...
      ...
    return list;
});
```

Map의 메서드를 잘 활용하지 못했는데, 이번 코딩 테스트를 준비하면서 유용한 기능들을 많이 알게 되었다.

``forEach()``를 통해 순회하면서 key와 value를 동시에 사용할 수 있다. 또한 ``computeIfAbsent()`` 혹은 ``computeIfPresent()``를 사용하면 특정 key의 존재 유무에 관련된 조건문 없이 작업을 지시할 수 있다.

특정 key가 없을 때, 단순히 새로운 value만 추가하는 것이 아니라 람다식을 통해 추가적인 작업을 수행할 수 있다. 이를 통해 코드를 상당히 축약할 수 있다.

<br>

---

## References

* [[JAVA] Stream 요소를 그룹핑해서 수집하기(Collectors.groupingBy)](https://cornswrold.tistory.com/387)
* [[Java8] sorted groupBy](https://devidea.tistory.com/58)
