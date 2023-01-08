---
title: "[Effective Java] Item 11. equals를 재정의하려거든 hashCode도 재정의하라"
excerpt: "서로 다른 인스턴스라면 해시코드도 서로 다르게 구현한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-26
last_modified_at: 2021-01-26
---

## 1. Object의 hashCode 명세

* equals 비교에 사용하는 정보가 변하지 않았다면 어플리케이션이 실행되는 동안 해당 객체의 hashCode는 항상 일관된 값을 반환해야 한다.
  * 어플리케이션이 다시 실행된다면 값이 변경되도 괜찮다.
* equals가 두 객체를 같다고 판단하면 두 객체의 hashCode는 똑같은 값을 반환한다.
* equals가 두 객체를 다르게 판단하더라도 두 객체의 hashCode가 다를 필요는 없다.
  * 단, 다른 객체에 대해 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

<br>

## 2. HashMap 예시

> Test.java

```java
Map<Car, String> carMap = new HashMap<>();
carMap.put(new Car(100), "KIA");
carMap.get(new Car(100)); //null이 반환된다.
```

두 객체가 재정의된 equals 연산 결과 논리적으로 동치일지라도 hashCode를 재정의하지 않으면 서로 다른 해시코드를 가진다. Hash 관련 컬렉션은 해시코드가 다른 엔트리끼리는 동치성 비교조차 시도하지 않도록 최적화되어 있다.

동치인 모든 객체들이 같은 해시코드를 반환하도록 하면 모든 객체가 하나의 해시테이블 버켓에 담겨 연결 리스트처럼 작동한다.

* 해시 테이블의 평균 수행 시간이 O(1)에서 O(n)으로 늘어나게 된다.
* 이상적인 해시 함수는 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

<br>

## 3. hashCode 재정의

equals 비교에 사용되는 필드들로 해시코드를 계산한다. 그 외의 필드를 해시코드 계산에 사용하면 일반 규약을 위배하게 된다.

### 3.1. Object의 hash 메서드

``Object.hash()``를 통해 해시코드를 계산하여 재정의한다.

* 입력 인수들을 담기 위한 배열이 생성되며, 박싱 및 언박싱 등의 작업이 추가로 발생할 수 있다.
* 직접 제작한 해쉬 함수보다 성능이 느릴 수 있다.

### 3.2. 해쉬 함수 직접 제작

> Instance.java

```java
@Override
public int hashCode() {
    int result = Short.hashCode(field1);
    result = 31 * reuslt + Integer.hashCode(field2);
    result = 31 * result + Integer.hashCode(field3);
    return result;
}
```

1. 필드의 값이 기본 타입이면 박싱 클래스의 hashCode를 사용해 계산한다.
2. 필드의 값이 참조 변수인 경우 표준형을 만들어 표준형의 hashCode를 사용해 계산한다.
3. 배열의 경우 핵심 원소들을 각각의 필드로 다루어 계산하되, 모든 원소들이 핵심 원소라면 ``Arrays.hashCode()``를 사용해 계산한다.
4. 참조 변수 필드가 null이거나 배열에 핵심 원소가 없다면 0 등의 상수를 이용하여 계산한다.

> String의 hashCode를 곱셈 없이 구현하면 철자가 같고 순서가 다른 문자열(아나그램)의 경우 해시코드가 모두 같아진다.

곱셈 계산 순서에 따라 값이 달라지기 때문에 비슷한 필드가 여러 개일 때 해쉬 효과가 높아진다. 곱할 숫자는 홀수이며 소수인 숫자를 사용한다.

<br>

## 4. 해시코드 캐싱

클래스가 불변이고 해시코드를 계산하는 비용이 크다면 캐싱 방식을 고려한다. 인스턴스가 만들어질 때 해시코드를 계산해두거나 지연 초기화 방식을 고려한다.

* 단, 지연 초기화 방식을 사용하려면 스레드에 안전하도록 코드를 구현해야 한다.

> Instance.java

```java
private int hashCode = 0; //인스턴스 생성시 필드를 초기화한다.

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(field1);
        result = 31 * reuslt + Integer.hashCode(field2);
        result = 31 * result + Integer.hashCode(field3);
    }
    return result;
}
```

* hashCode 필드의 초기값은 흔히 생성되는 객체의 해시코드와 달라야 한다.
* 핵심 필드를 누락해서는 안 된다.
  * 계산 속도는 빨라지지만 해시 품질이 나빠진다.
  * 수 많은 인스턴스들이 단 몇 개의 해시코드에 집중되어 해시 테이블의 속도가 선형적으로 느려지게 된다.
* hashCode가 반환하는 값의 생성 규칙을 API에 명시하지 않는다.
  * 클라이언트가 이 값에 의지하게 되면 추후에 더 나은 다른 해시 계산 방식을 적용(수정)하기 어려워진다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
