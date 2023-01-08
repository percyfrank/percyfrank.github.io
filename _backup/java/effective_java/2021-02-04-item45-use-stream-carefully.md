---
title: "[Effective Java] Item 45. 스트림은 주의해서 사용하라"
excerpt: "스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-04
last_modified_at: 2021-02-04
---

## 1. 스트림(Stream)

* 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
* 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
* 스트림 API는 다량의 데이터 처리 작업을 돕는다.

### 1.1. 스트림 파이프라인

* 스트림 파이프라인은 소스 스트림에서 시작해 종단 연산으로 끝난다.
  * 그 사이에 하나 이상의 중간 연산이 있을 수 있다.
  * 각 중간 연산은 스트림을 어떠한 방식으로 변환한다.
    * 한 스트림을 다른 스트림으로 변환하는데 타입이 전과 같거나 다를 수 있다.
* 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다.
* 스트림 파이프라인은 지연 평가된다.
  * 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.
* 스트림 파이프라인은 순차적으로 수행된다.
  * 병렬로 실행하려면 parallel 메서드를 호출해주어야 한다.

<br>

## 2. 주의사항

스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

> HybridAnagrams.java

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) { //도우미 메서드
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
* 반복문과 스트림을 적절히 조합하는 것이 나은 경우가 많다.
* 람다식에서 사용하는 매개변수 이름을 잘 지어야 가독성이 유지된다.
* 도우미 메서드를 적절히 활용하면 세부 로직을 분리함으로써 스트림 파이프라인의 가독성이 높아진다.
* char 스트림은 원소가 int이기 때문에 강제 형변환이 필요하므로 스트림을 삼가는 편이 낫다.
* 데이터가 파이프라인의 여러 단계를 통과할 때, 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어렵다.
  * 스트림 파이프라인은 한 값을 다른 값에 매핑하면 원래 값을 읽는 구조다.

반복문에서는 가능하지만 함수 객체에서는 불가능한 것들은 다음과 같다.

* 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고, 지역변수는 수정할 수 없다.
* 코드 블록과 달리 return, break, continue 등의 조건을 사용할 수 없다.
* 메서드 선언에 명시된 검사 예외를 던질 수 없다.

다음은 스트림에 어울리는 작업들이다.

* 원소들의 시퀀스를 일관되게 변환한다.
* 원소들의 시퀀스를 필터링한다.
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
* 원소들의 시퀀스를 컬렉션에 모은다.
* 원소들의 시퀀스에서 특정 조건에 만족하는 원소를 찾는다.

<br>

## 3. 평탄화

> Card.java

```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}

private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit ->
                    Stream.of(Rank.values())
                            .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```

* 두 집합의 데카르트 곱은 이중 반복문 혹은 평탄화를 통한 스트림으로 표현할 수 있다.
* ``flatMap()``은 스트림의 원소 각각을 하나의 스트림으로 맵핑한 다음, 그 스트림들을 다시 하나의 스트림으로 합친다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
