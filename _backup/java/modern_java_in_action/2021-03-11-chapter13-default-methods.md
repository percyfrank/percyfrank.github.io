---
title: "[Modern Java in Action] 13장. 디폴트 메서드"
excerpt: "인터페이스에 디폴트 메서드를 추가하게 되면 소스 호환성을 유지하면서 API를 업데이트할 수 있다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-11
last_modified_at: 2021-03-11
---

## 1. 진화하는 API

사람들이 대중적으로 사용하고 있는 라이브러리 API에 새로운 기능이 추가된다고 가정해보자. 해당 인터페이스를 구현하고 있는 기존의 구현 클래스들은 추가된 메서드를 반드시 구현해야 한다. 라이브러리 개발자는 해당 인터페이스를 사용하고 있는 클라이언트의 코드에 접근해 수정할 수 있는 권한이 없다. 새로운 기능이 추가되면 클라이언트 사이드의 코드에 번거로운 수정 작업이 필수적으로 수반된다.

### 1.1. 호환성의 종류

* 바이너리 호환성 : 변화가 생기더라도 에러 없이 잘 동작하는 기존의 바이너리 코드는 계속 링크할 수 있다.
  * 인터페이스에 새로운 메서드를 추가해도, 호출되지만 않으면 기존의 인터페이스 메서드는 문제 없이 동작하기 때문에 바이너리 호환성은 깨지지 않는다.
    * 그러나 어플리케이션을 재빌드하면 컴파일 에러가 발생한다.
* 소스 호환성 : 변화가 생기더라도 기존의 프로그램은 여전히 잘 컴파일된다.
  * 인터페이스에 새로운 메서드를 추가하면 소스 호환성이 깨진다.
    * 기존에 구현된 코드는 새로운 메서드를 구현하지 않아 재컴파일되지 않는다.
* 동작 호환성 : 변화가 생기더라도 기존의 프로그램은 동일한 입력에 대해 동일한 동작을 수행한다.
  * 인터페이스에 새로운 메서드를 추가해도, 새로운 메서드는 프로그램에서 절대로 호출되지 않기 때문에 동작 호환성은 깨지지 않는다.

<br>

## 2. 디폴트 메서드

Java 8 부터는 인터페이스가 정적 메서드 및 디폴트 메서드를 가질 수 있게끔 허용해줌으로써 진화하는 API로 인한 이슈를 유연하게 대처할 수 있다. 추상 메서드와 다르게 디폴트 메서드는 인터페이스에서 메서드 몸통까지 구현한다. 인터페이스를 구현 중이던 기존의 구현 클래스들은 별도의 오버라이딩이 없는 한 디폴트 메서드를 자동적으로 상속받는다. 즉, 인터페이스에 디폴트 메서드를 추가하게 되면 소스 호환성을 유지하면서 API를 업데이트할 수 있다.


> Resizable.java

```java
default void setRelativeSize(int wFactor, int hFactor) {
    setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
}
```

* 메서드 이름 앞에 default 키워드를 붙여준다.

### 2.1. 인터페이스 vs 추상 클래스

* 클래스는 단 하나의 추상 클래스만을 상속받을 수 있으나 인터페이스는 다중 구현이 가능하다.
* 추상 클래스는 인스턴스 변수를 통해 공통 상태를 강제할 수 있으나 인터페이스는 (상수를 제외한) 인스턴스 변수를 가질 수 없다.

<br>

## 3. 디폴트 메서드 활용 패턴

> Iterator.java

```java
interface Iterator<T> {

    boolean hasNext();

    T next();

    default void remove() {
        throw new UnsupportedOperationException();
    }
}
```

* 인터페이스의 메서드를 선택적으로 구현하도록 하는 경우 Java 8 이전까지는 불필요한 코드 관용구가 발생하곤 했다.
  * 해당 메서드를 구현하기 싫은 클래스들도 몸통이 텅 빈 메서드를 구현해야했기 때문이다.
* 디폴트 메서드의 도입을 통해 불필요한 코드 관용구가 사라지게 되었다.

> ArrayList.java

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, Serializable {
}
```

* ArrayList는 AbstractList, List, RandomAccess, Cloneable, Serializable, Iterator, Collection 등 총 7개 동작 타입을 다중 상속하는 효과를 가진다.
  * 여러 인터페이스로부터 디폴트 메서드를 상속받을 수 있기 때문이다.
* 인터페이스도 최소한으로 유지해야 불필요한 복잡성 없이 훌륭한 조합 효과를 누릴 수 있다.
  * 상속보단 조합을.

<br>

## 4. 해석 규칙

클래스는 부모 클래스 및 인터페이스들로부터 동일한 시그니처의 (디폴트) 메서드를 상속받을 수 있다. Java 컴파일러는 해석 규칙을 적용하여 어떤 메서드를 사용해야 할지 결정한다. 해석 규칙은 다음의 3단계로 구성되어 있다.

1. (상위) 클래스의 메서드가 인터페이스의 디폴트 메서드보다 우선순위가 항상 높다.
2. 가장 구체적인 서브 인터페이스의 우선순위가 그 다음으로 높다.
3. 여전히 선택이 애매한 경우 어떤 메서드를 사용할지 명시적으로 선언해야 한다.

> Main.java

```java
public class Main {

    interface A {

        default void hello() {
            System.out.println("hello from a");
        }
    }

    interface B {

        default void hello() {
            System.out.println("hello from b");
        }
    }

    class C implements A, B {

        void hello() {
            B.super.hello();
        }
    }

    public static void main(String[] args) {
        new C().hello();
    }
}
```

* ``B implements A``라면 B가 더 구체적이다는 뜻이다.
* 컴파일러가 해석 규칙을 적용해도 어떤 메서드를 사용해야할지 선택할 수 없는 경우 컴파일에 실패한다.
  * ``X.super.hello()``와 같이 어떤 메서드를 사용할지 명시한다.

> Diamond.java

```java
public class Diamond {

    interface A {

        public default void hello() {
            System.out.println("Hello from A");
        }
    }

    interface B extends A {
    }

    interface C extends A {
    }

    class D implements B, C {
    }

    public static void main(String... args) {
        new D().hello();
    }
}
```

* 현재의 경우 인터페이스 A의 디폴트 메서드가 호출된다.
* 인터페이스 B에만 인터페이스 A와 동일한 시그니처의 디폴트 메서드를 추가한다면 인터페이스 B의 디폴트 메서드가 호출된다.
  * 해석 규칙에 따라 B가 더 구체적이기 때문이다.
  * 인터페이스 B가 ``hello()``와 동일한 시그니처의 추상 메서드를 선언하면 이를 사용하되 클래스 D에서 구현해야 한다.
* 인터페이스 B와 C가 모두 디폴트 메서드를 선언한다면 B와 C 중 어떤 인터페이스의 메서드를 사용할지 명시해야 한다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
