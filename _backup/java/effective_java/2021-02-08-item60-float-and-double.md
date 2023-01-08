---
title: "[Effective Java] Item 60. 정확한 답이 필요하다면 float와 double은 피하라"
excerpt: "BigDecimal이 제공하는 여덟 가지 반올림 모드를 이용하여 반올림을 완벽히 제어할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. 부동소수점

float와 double 타입은 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 근사치로 계산하도록 설계되었다. 금융 관련 계산 등 정확한 결과가 필요할 때는 사용하면 안 된다.

<br>

## 2. BigDecimal

> BigDecimalChange.java

```java
public class BigDecimalChange {

    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal(".10");

        int itemsBought = 0;
        BigDecimal funds = new BigDecimal("1.00");
        for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
}
```

금융 계산에서는 int 혹은 long이나 BigDecimal을 사용해야 한다.

* BigDecimal은 기본 타입보다 쓰기 불편하고 느리지만 소수점 추적을 해준다.
  * 열여덟 자리 십진수를 넘는 단위의 수는 BigDecimal을 사용해야 한다.
* int나 long은 속도는 빠르지만 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
