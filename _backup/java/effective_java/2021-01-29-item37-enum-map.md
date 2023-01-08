---
title: "[Effective Java] Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라"
excerpt: "애플리케이션 개발자는 ordinal을 웬만해서는 사용하지 말아야 한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. ordinal 인덱싱

> Plant.java

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    public static void main(String[] args) {
        Plant[] garden; //있다고 가정
        Set<Plant>[] plantsByLifeCycleArr = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();
        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
     }
}
```

생애주기별로 식물들을 묶는 코드이다.

* 배열은 제네릭과 호환되지 않아 컴파일 경고가 발생한다.
* 배열의 각 인덱스가 어떤 의미인지 모르기 때문에 별도의 레이블이 필요하다.
* 열거 타입에 수정이 발생하면 ``ordinal()``이 잘못된 정수값을 반환할 수 있다.

<br>

## 2. EnumMap 활용

> Plant.java

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
```

열거 타입을 키로 사용하도록 설계된 Map 구현체인 EnumMap을 사용할 수 있다.

* 형변환이 타입 안전하다.
* 맵의 키 자체가 열거 타입이기 때문에 인덱스와 다르게 레이블이 필요없다.
* ``ordinal()``을 통해 배열 인덱스를 계산 과정에서 오류가 날 일이 없다.
* EnumMap 또한 내부에서 배열을 사용한다.
  * 내부 구현 방식을 Map 내부로 숨겨서 타입 안전성과 성능을 모두 얻었다.
* EnumMap의 생성자인 키 타입 Class 객체는 한정적 타입 토큰이다.

### 2.1. Stream 응용

> Plant.java

```java
Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```

반복문을 사용하지 않고 Stream과 groupingBy를 통해서도 코드를 짤 수 있다. 다만 배열을 사용한 버전은 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전은 해당 생애주기에 속하는 식물이 있을 때만 만든다.

> groupingBy 메서드 세부 설명은 [[Java] Stream 데이터를 groupingBy로 그룹핑해 Map으로 수집하기](https://xlffm3.github.io/java/map_using_collectors/) 포스팅을 참조하자.

### 2.2. 다차원 관계

> Phase.java

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
    }
}
```

* 상태와 전이 관련 맵핑 정보를 ``ordinal()``을 통해 2차원 배열로 표현할 수 있다.
  * 그러나 ``ordinal()``을 통해 배열 인덱스를 관리하는 것은 위험하다.
  * 새로운 상태와 전이 관련 상수가 추가되면 2차원 배열 코드 또한 일일이 수정해주어야 해서 Typo Safe하지 않다.
* 상태와 전이 관련 맵핑 정보의 다차원 관계 또한 EnumMap을 사용하여 표현할 수 있다.
  * 새로운 상수가 추가 되더라도 기존 로직은 변함없다.
* 실제 맵 내부 또한 배열로 구현되기 때문에 낭비되는 공간과 시간도 거의 없다.
* 명확하고 안전해서 유지보수하기 좋다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
