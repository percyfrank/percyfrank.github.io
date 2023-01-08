---
title: "[Effective Java] Item 85. 자바 직렬화의 대안을 찾으라"
excerpt: "시스템을 밑바닥부터 설계한다면 JSON이나 프로토콜 버퍼 같은 대안을 사용하자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 직렬화

객체 직렬화란 Java가 객체 그래프를 바이트 스트림으로 인코딩(직렬화)하고, 그 바이트 스트림으로부터 다시 객체를 재구성하는(역직렬화) 메커니즘이다. 개발자가 어렵지 않게 분산 객체를 만들 수 있다.

### 1.1. 취약점

근본적으로 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어려워진다. ``readObject()`` 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있다. 바이트 스트림을 역직렬화하는 과정에서 해당 타입들의 코드 전체가 공격 범위에 들어간다.

* 가젯(gadget)이란 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드이다.
* 가젯 체인을 구성하여 공격자가 기반 하드웨어의 네이티브 코드를 마음대로 실행할 수 있게 된다.
* 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하는 것으로 DOS 공격에 노출되기 쉽다.

> DeserializationBomb.java

```java
public class DeserializationBomb {

    public static void main(String[] args) throws Exception {
        System.out.println(bomb().length);
        deserialize(bomb());
    }

    static byte[] bomb() {
        Set<Object> root = new HashSet<>();
        Set<Object> s1 = root;
        Set<Object> s2 = new HashSet<>();
        for (int i = 0; i < 100; i++) {
            Set<Object> t1 = new HashSet<>();
            Set<Object> t2 = new HashSet<>();
            t1.add("foo"); // t1을 t2와 다르게 만든다.
            s1.add(t1);
            s1.add(t2);
            s2.add(t1);
            s2.add(t2);
            s1 = t1;
            s2 = t2;
        }
        return serialize(root);
    }
}
```

* 위 객체 그래프는 201개의 HashSet 인스턴스로 구성되며, 각각은 3개 이하의 객체 참조를 갖는다.
* 스트림의 전체 크기는 크지 않지만, HashSet 인스턴스를 역직렬화할 때 원소들의 해시코드를 계산하게 된다.
  * 반복문에 의해 객체 그래프 구조가 깊어져 ``hashCode()`` 메서드를 2^100 이상 호출하게 된다.

<br>

## 2. 대안

### 2.1. 크로스-플랫폼 구조화된 데이터 표현

직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것이다. 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘은 많이 존재한다. 해당 방식들은 Java 직렬화의 위험을 회피하면서 다양한 이점을 제공한다.

* 크로스-플랫폼 구조화된 데이터 표현은 Java 직렬화보다 훨씬 간단하다.
  * 객체 그래프를 직렬화 및 역직렬화하지 않고, 속성-값 쌍의 집합으로 구성된 간단하고 구조화된 데이터 객체를 사용한다.
  * 강력한 분산 시스템을 구축할 수 있다.
* JSON과 프로토콜 버퍼가 대표적이다.

### 2.2. 객체 역직렬화 필터링

레거시 시스템 때문에 Java 직렬화를 배제할 수 없을 때의 차선책은 신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것이다.

* 차선책으로 객체 역직렬화 필터링을 사용한다.
* 데이터 스트림이 역직렬화되기 전에 필터를 설치한다.
* 블랙리스트에 기록된 잠재적으로 위험한 클래스를 거부하거나, 화이트리스트에 기록된 안전한 클래스만 수용한다.
  * 블랙리스트 방식보다는 화이트리스트 방식을 추천한다.
* 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 사태를 보호해준다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
