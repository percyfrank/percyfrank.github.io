---
title: "[Modern Java in Action] 4장. 스트림 소개"
excerpt: "컬렉션이 supplier-driven이라면, 스트림은 consumer-driven의 성격을 지닌다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-02-21
last_modified_at: 2021-02-21
---

## 1. 스트림(Stream)이란?

스트림 API는 선언적인 방식으로 컬렉션 데이터를 조작한다. 사용자는 데이터 조작에 필요한 기능을 장황한 코드로 구현하지 않고, SQL처럼 쿼리로서 표현할 수 있다. 또한 스트림은 병렬 처리성이 우수하다.

컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조이다. 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다. 그리고 컬렉션에서 값을 꺼내서 연산할 경우, 모든 연산 횟수만큼 탐색해야 한다. 반면, 스트림은 사용자가 요청하는 값만 추출하여 계산하는 게으른 평가 특징이 있다.

> StreamBasic.java

```java
public static List<String> getLowCaloricDishesNamesInJava7(List<Dish> dishes) {
    List<Dish> lowCaloricDishes = new ArrayList<>();
    for (Dish d : dishes) {
        if (d.getCalories() < 400) {
          lowCaloricDishes.add(d);
        }
    }
    List<String> lowCaloricDishesName = new ArrayList<>();
    Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
        @Override
        public int compare(Dish d1, Dish d2) {
          return Integer.compare(d1.getCalories(), d2.getCalories());
        }
    });
    for (Dish d : lowCaloricDishes) {
        lowCaloricDishesName.add(d.getName());
    }
    return lowCaloricDishesName;
}
```

> StreamBasic.java

```java
public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.parallelStream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
}
```

스트림의 특징은 다음과 같다.

* 코드를 선언적인 방식으로 작성할 수 있다.
  * 특정한 기능(작업)을 어떻게(How) 구현(수행)할 지보다는 무엇을(What) 달성(수행)하고자 하는지를 서술하게 된다.
    * 칼로리가 낮은 음식을 필터링하는 기능을 어떻게(반복문과 조건문의 조합 등) 구현할지 보다는, **내가 원하는 기준에 맞춰 필터링해줘**라고 얻고자 하는 목적을 명시한다.
    * Stream API가 ``filter()``, ``map()``, ``limit()`` 등의 기능을 제공하고 있기 때문에 사용자는 해당 기능들의 구현을 신경쓰지 않아도 된다.
  * 복사 붙여넣기 등 코드 중복 없이 동적 파라미터화(람다)를 통해 변화하는 요구사항에 유연하게 대처할 수 있다.
  * 쿼리 형태의 명령은 순차와 병렬 처리를 모두 지원한다.
* 여러 연산들을 체이닝하여 복잡한 데이터 처리 파이프라인을 구성할 수 있다.
  * 가독성이 우수하다.
* Stream API의 연산들은 특정 스레딩 모델이 의존하지 않아서, 내부 구현은 싱글 스레드뿐만 아니라 멀티코어 구조에서도 최적화될 수 있다.
  * 스레드나 락 등 복잡한 동시성 제어에 크게 신경쓸 필요가 없어진다.

### 1.1. 용어 정리

스트림(Stream)은 데이터 처리 연산을 지원하는 소스의 원소 시퀀스이다.

* 원소 시퀀스.
  * 스트림은 컬렉션처럼 순서가 있는 특정한 타입의 원소들에 대한 인터페이스를 제공한다.
  * 컬렉션은 자료구조로서 데이터를 저장하고 접근하는데 시공간 복잡도를 가진다.
  * 반면 스트림은 원소들에 대해 특정한 연산(filter, map 등)을 표현한다.
* 소스.
  * 컬렉션, 배열, 입출력 리소스 등 데이터를 제공하는 소스에서 스트림이 소비된다.
  * 순서가 있는 컬렉션은 그 스트림 또한 동일한 순서를 가진다.
* 데이터 처리 연산.
  * 스트림은 DB 및 함수형 프로그래밍 언어와 같은 연산들을 지원한다.
  * 순차 및 병렬 처리를 지원한다.

<br>

## 2. Stream vs Collection

스트림과 컬렉션의 큰 차이는 언제 원소들이 연산되느냐이다. 컬렉션은 인 메모리 자료 구조로서, 해당 자료 구조가 현재 가지고 있는 원소들을 모두 **쥐고 있는** 상태이다. 컬렉션의 원소들은 컬렉션에 추가되기 전에 연산되어야 한다.

* 컬렉션에 원소를 추가하거나 삭제할 때, 컬렉션의 모든 원소들은 메모리에 저장되어 있는 상태이다.
  * DVD는 모든 데이터를 쥐고 순차적으로 읽어 영상을 보여준다.

반면, 스트림은 개념적으로 고정된 자료 구조로서 원소를 추가 및 삭제를 할 수 없다. 다만 스트림의 원소들은 사용자가 요청하는 필요한 시점에(on demand) 연산될 수 있다. 따라서 스트림은 구성되는 시점이 지연적인 컬렉션이다.

* 영상 스트리밍 서비스는 모든 스트림들이 연산되지 않은 상태이더라도, 먼저 연산된 스트림의 앞 부분들로부터 데이터를 읽어 실시간으로 영상을 보여준다.

컬렉션이 supplier-driven이라면, 스트림은 consumer-driven의 성격을 지닌다. 이를 극명하게 보여주는 사례는 **무한 스트림**이다. 무한한 크기의 컬렉션을 생성하려면 무한 반복문으로 인해 사용자는 이를 결코 사용할 수 없다. 그러나 스트림은 무한한 크기의 스트림을 생성할 수 있으며 사용자는 이를 사용할 수 있다.

* Google을 통해 검색할 때, 서버는 모든 검색 결과 정보를 컬렉션에 담아 제공하지 않는다.
  * 검색 결과의 양이 많은 경우 클라이언트가 받아보기 까지 시간이 오래 소요된다.
  * 상위 n개의 검색 결과 스트림 원소들만 보여주고 사용자는 더 보기 버튼을 눌러 추가 데이터를 요청한다.
  * 서버는 사용자의 추가 요청이 있을 때마다 on demand로 추가 n개의 스트림 원소들을 연산하여 검색 결과를 제공한다.

데이터 소스로부터 열린 스트림은 한 번 소비되면 닫혀서 사용할 수 없다. 스트림을 또 순회하려거든 데이터 소스로부터 새로운 스트림을 받아야 한다.

### 2.1. 내부 순회 vs 외부 순회

컬렉션은 for 문이나 Iterator 등의 명시적인 수단으로 외부 순회하지만, 스트림은 내부 순회한다. 외부 순회는 기본적으로 단일 스레드 상의 순차 순회이다. 병렬화 및 동시성 등의 작업은 사용자의 추가적인 자가 관리(synchronized, lock, thread 등)가 필요하다.

반면 내부 순회를 사용하면 원소 처리 작업이 병렬 (혹은 작업을 최적화하는 다른 순서)로 쉽게 수행될 수 있다. Stream 라이브러리의 내부 순회는 하드웨어에 맞춰서 병렬화 구현 및 데이터 표현 등을 자동 선택한다.

### 2.2. 연산

스트림 연산은 크게 중간 연산과 종단 연산으로 구분된다.

#### 2.2.1. 중간 연산

> Main.java

```java
public static void main(String[] args) {
    List<String> names = menu.stream() //menu의 원소는 총 10개
          .filter(dish -> {
              System.out.println("filtering " + dish.getName());
              return dish.getCalories() > 300;
          })
          .map(dish -> {
              System.out.println("mapping " + dish.getName());
              return dish.getName();
          })
          .limit(3)
          .collect(toList());
    System.out.println(names);
    /*
    출력 결과
    filtering:pork
    mapping:pork
    filtering:beef
    mapping:beef
    filtering:chicken
    mapping:chicken
    [pork, beef, chicken]
    */
}
```

* 중간 연산은 스트림을 소비하지 않으며 리턴 타입으로 또 다른 스트림을 반환한다.
  * 쿼리의 여러 연산들이 체이닝을 통해 파이프라인을 구성할 수 있도록 돕는다.
  * ``filter()``는 Predicate를, ``map()``은 Function을, ``sorted()``는 Comparator를 인자로 받는다.
* 종단 연산이 호출되기 전 까지 데이터 처리를 수행하지 않는 지연 속성을 가지고 있다.
  * 중간 연산들은 종단 연산에 의해 하나로 합쳐져 처리된다.
  * 위 스트림의 원소는 총 10개지만 중간 연산들의 출력문은 3개의 원소만 출력한다.
    * 종단 연산 호출 후 ``limit()`` 등의 중간 연산들이 합쳐진 다음 출력문이 invoke되기 때문이다.

#### 2.2.2. 종단 연산

종단 연산은 스트림을 소비하여 파이프라인으로부터 결과를 생성한다. 결과는 컬렉션이나 일반 자료형 및 void 등 스트림 값이 아닌 것을 의미한다.

* ``forEach()``는 void를 반환하고 각 스트림 요소들에게 람다식을 적용하는 종단 연산이다.
* 중간 연산-종단 연산의 조합을 통해 생성하는 파이프라인은 [빌더 패턴](https://xlffm3.github.io/java/item2-builder/)과 유사하다.
  * 객체 생성에 필요한 필드를 메서드 체이닝(중간 연산)으로 대입한다.
  * 마지막에 종단 연산격인 ``build()``를 호출하여 객체를 최종적으로 생성해낸다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
