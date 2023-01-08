---
title: "[Effective Java] Item 33. 타입 안전 이종 컨테이너를 고려하라"
excerpt: "컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 다양한 타입 매개변수를 다룰 수 있다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-29
last_modified_at: 2021-01-29
---

## 1. 타입 안전 이종 컨테이너 패턴

제네릭은 ``Set<E>``, ``Map<K,V>``와 같이 단일 원소 컨테이너에도 사용된다. 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다. 단일 원소 컨테이너 패턴보다 더 유연함이 필요한 경우 타입 안전 이종 컨테이너 패턴을 사용한다.

* 타입 안전 이종 컨테이너 패턴이란?
  * Set이나 Map 등의 컨테이너 대신 키를 타입 매개변수화한다.
  * 컨테이너에서 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.
  * 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해준다.
* 타입 안전 이종 컨테이너 패턴을 이용하면 타입 매개변수의 수가 고정된 단일 원소 컨테이너의 제약이 사라진다.

<br>

## 2. 예시

> Favorites.java

```java
public class Favorites {

    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }

    public static void main(String[] args) {
        Favorites f = new Favorites();

        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);

        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
    }
}
```

* class의 리터럴 타입(클래스)은 Class가 아닌 ``Class<T>``이다.
  * ``String.class``의 타입은 ``Class<String>``이다.
* 컴파일타입 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴(객체)을 **타입 토큰(Type Token)**이라고 한다.

Favorites는 타입 안전하다. 특히 Map이 아니라 Key가 와일드카드 타입이기 때문에 모든 키가 서로 다른 매개변수화 타입이 될 수 있다. 문제는 Value 타입이 Object이기 때문에 타입 정보의 링크가 끊어져서 별도로 캐스팅 작업이 수반된다.

> Class.java

```java
public class Class<T> {
     T cast(Object obj);
}
```

Class의 ``cast()``는 객체 참조를 class 객체가 가리키는 타입으로 동적 변환한다. Class 클래스가 제네릭이기 때문에 ``cast()`` 메서드의 반환 타입은 호출한 class 객체의 타입 매개변수와 같다. T로 비검사 형변환하는 손실 없이도 Favorites를 타입 안전하게 만든다.

* 캐스팅에 실패하면 ClassCastException이 발생한다.

### 2.1. 제약

#### 2.1.1. 첫 번째 제약

악의적인 클라이언트가 Class 객체를 제네릭이 아닌 로 타입으로 넘기면 Favorites의 타입 안전성이 깨진다.

> Main.java

```java
favorites.put((Class) Integer.class, "Invalid Type");
int value = favorites.getFavorite(Integer.class); //ClassCastException 발생
```

* 컴파일은 가능하지만 비검사 경고가 발생하고 런타임시 형변환 실패로 예외가 발생한다.

> Main.java

```java
HashSet<Integer> set = new HashSet<>();
((HashSet) set).add("문자열");
```

* 이 코드는 컴파일도 되고 동작도 하게 된다.

> Favorites.java

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

* 이러한 제약을 극복하려면 값을 저장할 때 Class의 ``cast()`` 메서드를 활용한다.
* instance의 타입이 type으로 명시된 타입과 같은지 확인하게 된다.

#### 2.1.1. 두 번째 제약

Favorites 클래스의 두 번째 제약은 실체화 불가 타입에는 사용할 수 없다는 점이다.

* String이나 String[]은 저장 가능해도, ``List<String>``은 저장이 불가능하다.
* ``List<String>``과 ``List<Integer>`` 모두 같은 ``List.class``라는 객체를 공유한다.
  * 슈퍼 타입 토큰이라는 우회로가 있지만 안전하지는 않다.

<br>

## 3. 한정적 타입 토큰

Favorites는 사용 타입 토큰이 비한정적이다. 허용하는 타입을 제한하고 싶으면 한정적 타입 토큰을 사용한다.

> PrintAnnotation.java

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

* 문제는 ``Class<?>``같은 비한정적인 객체를 위와 같이 한정적 타입 토큰을 받는 메서드에 전달할 때이다.
* 객체를 ``Class<T extends Annotation>``으로 형변환할 수도 있지만 비검사이므로 컴파일 경고가 발생한다.

> PrintAnnotation.java

```java
public class PrintAnnotation {

    static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
        Class<?> annotationType = null; //비한정적 타입 토큰
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        return element.getAnnotation(annotationType.asSubclass(Annotation.class));
    }
}
```

* ``asSubclass()`` 메서드는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다.
  * 형변환된다는 것은 이 클래스가 인수로 명시된 클래스의 하위 클래스라는 뜻이다.
* 이 방법을 사용하면 오류나 경고없이 컴파일된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
