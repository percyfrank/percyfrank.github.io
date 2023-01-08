---
title: "[Effective Java] Item 65. 리플렉션보다는 인터페이스를 사용하라"
excerpt: "리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-09
last_modified_at: 2021-02-09
---

## 1. 리플렉션

리플렉션 기능을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.

* Class 객체가 주어지면 클래스의 생성자, 메서드, 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다.
  * 해당 인스턴스들로 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.
  * 해당 인스턴스들로 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수 있다.
* 리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.
* 런타임에 존재하지 않는 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다.
  * 버전이 여러 개 존재하는 외부 패키지를 다룰 때 유용하다.
  * 가동할 수 있는 최소한의 환경(가장 오래된 버전)만을 컴파일한 후, 이후 버전의 클래스와 메서드 등은 리플렉션으로 접근한다.
    * 복잡한 특수 시스템을 개발할 때 유용하다.

### 1.1. 단점

리플렉션은 아주 제한된 형태로만 사용해야 단점을 피하고 이점만 취할 수 있다.

* 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.
  * 예외 검사도 마찬가지다.
  * 리플렉션을 써서 존재하지 않는 혹은 접근할 수 없는 메서드를 호출하려 시도할 때 예외를 잘 처리하지 않으면 런타임 오류가 발생한다.
* 리플렉션을 이용하면 코드가 장황해진다.
* 리플렉션을 통한 메서드 호출은 일반 호출보다 느리다.

컴파일타임에 알 수 없는 클래스는 리플렉션을 통해 사용할 수밖에 없다. 그러나 **리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스를 이용할 때는 컴파일타임에 알 수 있는 인터페이스나 상위 클래스로 참조해 사용하라.**

<br>

## 2. 예제

> ReflectiveInstantiation.java

```java
public class ReflectiveInstantiation {

    public static void main(String[] args) {
        // 클래스 이름을 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
        try {
            cl = (Class<? extends Set<String>>)  // 비검사 형변환!
                    Class.forName(args[0]);
        } catch (ClassNotFoundException e) {
            fatalError("클래스를 찾을 수 없습니다.");
        }

        // 생성자를 얻는다.
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
        }

        // 집합의 인스턴스를 만든다.
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("생성자에 접근할 수 없습니다.");
        } catch (InstantiationException e) {
            fatalError("클래스를 인스턴스화할 수 없습니다.");
        } catch (InvocationTargetException e) {
            fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
        } catch (ClassCastException e) {
            fatalError("Set을 구현하지 않은 클래스입니다.");
        }

        // 생성한 집합을 사용한다.
        s.addAll(Arrays.asList(args).subList(1, args.length));
        System.out.println(s);
    }

    private static void fatalError(String msg) {
        System.err.println(msg);
        System.exit(1);
    }
}
```

* Set\<String\> 인터페이스의 인스턴스를 리플렉션으로 생성한다.
* 런타임에 예외가 많이 발생한다.
  * 리플렉션 없이 생성했다면 컴파일타임에 잡을 수 있는 예외들이다.
* 코드가 장황해진다.
  * 리플렉션 예외 각각을 잡는 대신 ReflectiveOperationException이라는 모든 리플렉션 예외의 상위 코드를 잡으면 코드를 줄일 수 있다.
* 첫 번째 인수로 지정한 클래스가 무엇이냐에 따라 출력되는 순서는 달라진다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
