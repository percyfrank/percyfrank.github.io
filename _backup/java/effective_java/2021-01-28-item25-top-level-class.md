---
title: "[Effective Java] Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라"
excerpt: "하나의 소스 파일에 여러 톱레벨 클래스를 담아야 한다면 정적 멤버 클래스를 고려한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-28
last_modified_at: 2021-01-28
---

## 1. 톱레벨 클래스

> Example.java

```java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

소스 파일에 하나에 톱레벨 클래스가 여러 개 정의되어 있더라도 컴파일은 가능하다. 그러나 소스 파일의 톱레벨 클래스와 이름이 중복되는 클래스가 외부에 존재하는 경우, 소스 파일의 순서에 따라 컴파일 결과가 달라질 수 있다.

> Test.java

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

여러 톱레벨 클래스를 하나의 소스 파일에 담아야 한다면 정적 멤버 클래스를 고려하라. 그런 것이 아니라면 **소스 파일 하나에는 반드시 톱레벨 클래스(인터페이스)를 하나만 담자.**

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
