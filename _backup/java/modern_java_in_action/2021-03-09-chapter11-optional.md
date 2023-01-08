---
title: "[Modern Java in Action] 11장. null 대신 Optional 클래스"
excerpt: "null이 존재할 수 있는 곳에 방어적 검사 코드를 일일이 삽입하는 것은 생산성 하락으로 이어진다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-09
last_modified_at: 2021-03-09
---

## 1. NullPointerException

> Insurance.java

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

* Person은 Car를, Car는 Insurance를, Insurance는 String을 포함하고 있는 구조다.
* 만약 Person이나 Car 혹은 Insurance 객체가 값이 없는 null이라면 NullPointerException이 발생한다.

> Insurance.java

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

* NPE를 피하기 위한 방어적 검사는 조건문이 중첩되는 등 가독성이 심각하게 하락한다.
* null이 존재할 수 있는 곳에 이러한 방어적 검사 코드를 일일이 삽입하는 것은 생산성 하락으로 이어진다.
* 특정 상황에서 null 값을 표현하는 것이 옳바른지 고민하지 않고 방어 코드를 삽입하는 행위는 버그를 수정하는 것이 아니라 숨기는 것에 불과하다.

### 1.1. null의 문제점

* NullPointerException을 유발하는 에러의 근원이다.
* 방어적 검사 코드의 중첩으로 인해 가독성이 하락한다.
* null 자체에는 어떠한 의미가 존재하지 않다.
  * 값이 없음을 표현하는데 부적절하다.
* 개발자에게 포인터를 숨기려는 Java 언어의 철학에 위배된다.
* 타입 시스템에 큰 결함을 만든다.
  * null은 아무런 정보나 타입을 가지지 못하며 어떠한 참조 타입에도 대입될 수 있다.
  * null이 시스템의 다른 부분에 전파되면 해당 null이 원래 의미하던 바를 파악하기 어려워진다.

### 1.2. null에 대한 다른 언어들의 대안

* Groovy는 ?.,로 표현되는 safe navigator 연산자를 통해 잠재적인 null 값들에 대한 정보를 제공한다.
  * ``def carInsuranceName = person?.car?.insurance?.name``
  * 객체 참조에 null을 할당함으로써 특정한 값을 가지고 있지 않을 가능성을 시사한다.
* Haskell은 Maybe 타입을 도입하였으며 선택적인 값을 필수적으로 캡슐화한다.
  * Maybe는 주어진 타입의 값을 가지고 있거나 없음을 표현한다.
* Scala는 Option[T]를 통해 T 타입의 값이 존재 혹은 부재를 캡슐화한다.
  * 사용자에게 값이 존재하는지 명시적으로 체크하게 함으로써 null 체크를 강제화한다.

<br>

## 2. Optional

Optional 클래스는 선택적인 값을 캡슐화한다. 특정 필드의 값이 존재하지 않을 수 있는 경우 null을 할당하는 대신 Optional을 사용함으로써 의미를 명확하게 표현할 수 있다. 어플리케이션에서 일관되게 Optional을 사용한다면, 특정 값이 존재하지 않을 때 원인이 **의도된 것**인지 **알고리즘이나 데이터에서 비롯된 버그**인지 명확한 구분이 생긴다. Optional은 값이 선택적임을 시사함으로써 이해하기 쉬운 API 작성을 돕는다. 사용자가 값의 부재를 다루기 위해 Optional 래퍼를 벗기도록 강제한다.

* Car 객체는 Insurance가 없을 수 있기 때문에 Insurance 필드를 Optional로 선언한다.
* Insurance 객체는 이름이 항상 존재하기 때문에 String 필드를 Optional 없이 선언한다.

> Person.java

```java
public class Person {
    private Car car;

    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

* 그러나 값이 없을 수도 있음을 시사하기 위해 고안되었을 뿐, 필드 타입으로 사용하기 위해 고안된 것은 아니다.
  * Optional은 Serializable 인터페이스를 구현하지 않아 직렬화가 불가능하다.
* 직렬화가 필요한 도메인 객체라면 필드 타입을 Optional로 선언하지 않고, getter 메서드에서 해당 필드를 Optional로 반환한다.

<br>

## 3. Optional을 사용하는 패턴

> Car.java

```java
Optional<Car> optCar = Optional.empty();
Optional<Car> optCar = Optional.of(car);
Optional<Car> optCar = Optional.ofNullable(car);
```

* 값이 없는 경우 ``empty()``를 사용한다.
* ``of()``에 null 값이 들어가게 되면 그 즉시 NullPointerException이 발생한다.
* ``ofNullable()``은 해당 Optional 객체가 null 값을 포함할 수 있음을 뜻한다.
  * 타입 객체가 null인 경우 결과 Optional 객체는 empty이다.

> Car.java

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

* Stream과 비슷하게 ``map()``을 통해 Optional에 담긴 값에 인자로 주어진 Function을 적용하여 변환한다.
  * Optional이 비어있다면 아무 일도 일어나지 않는다.

### 3.1. 평탄화

> Car.java

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar)
                                .map(Car::getInsurance)
                                .map(Insurance::getName);
```

* ``map()``을 통해 값을 변경했을 때 변경된 값이 Optional이라면 Optional\<Optional\<...\>\> 구조가 된다.
  * 위 코드는 컴파일이 되지 않는다.

> Car.java

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
            .flatMap(Car::getInsurance)
            .map(Insurance::getName)
            .orElse("Unknown");
}
```

* 마찬가지로 Stream과 유사한 ``flatMap()``을 통해 Optional 값에 대한 평탄화 작업을 진행할 수 있다.
* Optional이 비어있다면 마찬가지로 함수가 적용되지 않고 비어있는 Optional이 그대로 파이프라인을 통해 전달된다.

### 3.2. Optional 스트림

> Car.java

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
            .map(Person::getCar)
            .map(optCar -> optCar.flatMap(Car::getInsurance))
            .map(optIns -> optIns.map(Insurance::getName))
            .flatMap(Optional::stream)
            .collect(toSet());
}
```

* 컬렉션을 스트림으로 다룰 때 원소가 Optional 값으로 맵핑하는 경우 Stream\<Optional\<...\>\> 형태가 도출된다.
* Java 9는 Optional의 ``stream()`` 메서드를 통해 비어있지 않은 Optional들에 한해서 값을 꺼내 Stream 형태로 반환한다.
  * ``flatMap()``을 통해 Optional 값들에 대해 평탄화를 진행해준다.

> Car.java

```java
Stream<Optional<String>> stream = ...
Set<String> result = stream.filter(Optional::isPresent)
                           .map(Optional::get)
                           .collect(toSet());
```

* 동일한 효과를 보여준다.

### 3.3. 기타

> Insurance.java

```java
optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

* ``get()``, ``orElse()``, ``orElseGet()``, ``orElseThrow()``, ``ifPresent()`` 등.
* ``or()`` ``orElseGet()``과 유사하지만 값이 존재하는 경우 Optional에서 값을 꺼내지 않고 Optional 객체 자체를 반환한다.
* Stream과 유사하게 Predicate를 받는 ``filter()`` 메서드를 사용하여 조건에 부합하는 Optional 객체를 필터링할 수 있다.
  * 비어있는 Optional 객체에 ``filter()``를 적용하면 똑같이 비어있는 객체가 반환된다.
  * 값이 존재하더라도 Predicate가 false를 반환하면 비어있는 객체를 반환한다.

<br>

## 4. 실사용 예제

Collection API는 Optional을 사용하지 않아 null을 반환하는 메서드들이 많이 존재한다. Map에서 특정 Key에 해당하는 Value를 찾을 때 값이 없을 경우 Optional로 감싸면 null 체크를 위한 방어적 코드를 생략할 수 있다.

> Utility.java

```java
public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        return Optional.empty();
    }
}
```

* Java API는 null 반환대신 예외를 발생시키는 경우가 많지만 간단한 유틸리티 클래스를 통해 Optional한 값을 반환하도록 강제할 수 있다.

> Duration.java

```java
public int readDuration(Properties props, String name) {
    String value = props.getProperty(name);
    if (value != null) {
        try {
            int i = Integer.parseInt(value);
            if (i > 0) {
                return i;
            }
        } catch (NumberFormatException nfe) {
        }
    }
    return 0;
}
```

> Duration.java

```java
public int readDuration(Properties props, String name) {
    return Optional.ofNullable(props.getProperty(name))
            .flatMap(Utility::stringToInt)
            .filter(i -> i > 0)
            .orElse(0);
}
```

### 4.1. 기본 자료형 특화 Optional

Optional 또한 불필요한 오토박싱을 방지하는 기본 자료형 특화 형태를 제공한다. 그러나 해당 Optional들은 ``map()``, ``flatMap()``, ``filter()`` 등의 유용한 기능이 누락되어서 유연하게 사용하기 어렵기 때문에 사용을 지양한다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
