---
title: "[Effective Java] Item 40. @Override 애너테이션을 일관되게 사용하라"
excerpt: "여러분이 실수했을 때 컴파일러가 바로 알려준다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-30
last_modified_at: 2021-01-30
---

## 1. 오버로딩

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }
}
```

* equals의 매개 변수는 Object이지만, 위 예제는 타입이 다른 매개변수를 명시하고 있다.
  * 오버라이딩(재정의)가 아닌 오버로딩(다중 정의)가 되었다.
  * 잘못된 equals로 런타임 중 원하지 않는 결과가 도래할 수 있다.
* @Override 애너테이션을 재정의 할 메서드에 붙였더라면 컴파일 오류가 발생한다.
  * 잘못한 부분을 지적해주기 때문에 바로 잡을 수 있다.

<br>

## 2. @Override

상위 클래스의 메서드를 재정의하려는 모든 메서드는 @Override 애너테이션을 붙이자.

* 구체 클래스에서 상위 추상 클래스의 추상 메서드를 재정의할 때는 굳이 달 필요는 없다.
  * 구현하지 않으면 컴파일러가 경고하기 때문이다.
* 그러나 인터페이스든 추상 클래스든 상위 클래스를 재정의하는 메서드에는 일관되게 애너테이션을 붙이는 것이 좋다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
