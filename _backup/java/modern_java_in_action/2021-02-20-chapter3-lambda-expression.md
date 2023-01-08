---
title: "[Modern Java in Action] 3장. 람다 표현식"
excerpt: "익명 클래스처럼 불필요하고 장황한 관용구를 작성할 필요가 없다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-02-20
last_modified_at: 2021-02-20
---

## 1. 람다 표현식(Lambda Expression)

람다 표현식은 전달될 수 있는 익명 함수를 간결하게 표현하는 방식이다. 람다 표현식의 특징은 다음과 같다.

* 메서드와 달리 명시적인 이름이 없는 익명이다.
* 메서드와 달리 특정 클래스와 관련이 없기 때문에 **함수**라고 한다.
  *  파라미터, 몸체, 리턴 타입, 발생 가능 예외 등의 특성은 메서드와 동일하다.
* 다른 메서드의 인수로 전달되거나 변수에 저장될 수 있다.
* 익명 클래스처럼 불필요하고 장황한 관용구를 작성할 필요가 없다.

> Filter.java

```java
Collections.sort(list, new Comparator<Apple>() {
    @Override
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }  
});

//익명 클래스를 람다로 리팩토링

Collections.sort(list, (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

* 화살표 기준으로 좌측은 파라미터 목록이며, 우측은 람다 몸체이다.
* 람다 몸체 표현식 자체가 람다의 리턴 값으로 간주된다.

### 1.1. Syntax

> Syntax.java

```java
(parameters) -> expression
(parameters) -> { statements; }

() -> "Raoul"
() -> { return "Raoul"; }
```

* 단순한 수식에는 괄호가 필요 없지만 구문의 경우 괄호가 필요하다.
* 람다 표현의 몸체가 비어있는 경우 ``() -> {}``와 같이 표현한다.
* ``process(() -> System.out.println("What?"));`` 같은 표현은 예외적으로 허용한다.
  * Java 언어 명세서에 단일 void 메서드는 괄호를 감싸지 않아도 된다는 예외 규칙이 존재한다.

<br>

## 2. 함수형 인터페이스

함수형 인터페이스란 **하나의 추상 메서드**를 명시하고 있는 인터페이스이다. Predicate, Comparator, Runnable 등이 함수형 인터페이스에 속한다. 함수형 인터페이스는 추상 메서드를 하나만 가지고 있는한 디폴트 메서드는 제한없이 보유할 수 있다.

* @FunctionalInterface 애너테이션은 해당 인터페이스가 함수형 인터페이스임을 문서화한다.
  * 오직 한 개만의 추상 메서드를 가지고 있는지 컴파일러가 체크해준다.

람다 표현식을 통해 함수형 인터페이스의 추상 메서드를 인라인으로 구현할 수 있으며, 전체 표현식을 함수형 인터페이스의 구현체(인스턴스)로 다룬다. Java는 언어의 복잡성을 고려하여 함수형 인터페이스를 사용하는 곳에서만 람다를 사용할 수 있도록 제한한다.

### 2.1. 함수 디스크립터(Function Descriptor)

함수형 인터페이스의 추상 메서드의 시그니처는 람다 표현식의 시그니처를 의미한다. 이러한 추상 메서드를 함수 디스크립터라 칭한다.

<br>

## 3. 실행 어라운드 패턴(Execute Around Pattern)

> ExecuteAroundPattern.java

```java
public static String processFileLimited() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
        return br.readLine();
    }
}
```

* DB나 파일 등을 다루는 자원 프로세싱은 작업을 실행하기 전에 준비하는 코드(자원 할당)와 작업 실행 후 완료하는 코드(자원 해제)를 수반한다.
* 위 코드는 1줄만을 읽어 반환하지만 요구사항이 변화할 때 유연하게 대처하기 어렵다.

> BufferedReaderProcessor.java

```java
public interface BufferedReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```

> ExecuteAroundPattern.java

```java
public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
      return p.process(br);
    }
}
```

> Main.java

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

<br>

## 4. 표준 함수형 인터페이스

다양한 람다 표현식을 사용하기 위해서는 다양한 함수 디스크립터를 가진 함수형 인터페이스가 필요하다. Java API는 자주 쓰이는 함수 디스크립터들의 표준 함수형 인터페이스들을 제공하고 있다.

* ``UnaryOperator<T>`` (함수 시그니처 ``T apply(T t)``)
* ``BinaryOperator<T>`` (함수 시그니처 ``T apply(T t1, T t2)``)
* ``Predicate<T>`` (함수 시그니쳐 ``boolean test(T t)``)
* ``Function<T, R>`` (함수 시그니쳐 ``R apply(T t)``)
* ``Supplier<T>`` (함수 시그니쳐 ``T get()``)
* ``Consumer<T>`` (함수 시그니쳐 ``void accept(T t)``)

### 4.1. 변형

지네릭스의 타입 매개변수는 레퍼런스 타입만 허용된다. Java는 오토박싱 메커니즘을 통해 기본 자료형과 해당 자료형의 레퍼런스 타입(래퍼 클래스)간의 박싱 및 언박싱을 자동화해준다. 박싱된 값들은 힙 메모리에 저장되기 때문에 기본 자료형에 비해 단점이 많다.

* 더 많은 메모리를 사용하며, 래핑된 기본 값을 가져오는데 추가적인 메모리 Lookup을 필요로 한다.
* 기본 자료형과 래핑 클래스를 섞어 사용하면 오토박싱으로 인해 성능이 저하된다.

표준 함수형 인터페이스는 기본 자료형을 위한 변형 버전을 제공하고 있으며, 이를 통해 불필요한 오토박싱을 피한다.

* 기본 인터페이스는 기본 타입인 int, long, double 용으로 각 3개씩 변형이 생겨난다.
  * IntPredicate, LongBinaryOperator 등.
  * Function은 LongFunction\<int[]\> 같이 반환 타입만 매개변수화한다.
* Function 인터페이스는 기본 타입을 반환하는 변형이 총 9개 있다.
  * 입력과 결과 타입이 모두 기본 타입이면 접두어에 ``SrcToResult``를 사용한다.
    * LongToIntFunction 등.
  * 입력이 객체 참조이고 결과가 기본 타입이면 접두어에 ``ToResult``를 사용한다.
    * ToLongFunction\<int[]\> 등.
* Predicate, Function, Consumer는 인수를 2개씩 받을 때 접두어에 ``Bi``를 사용한다.
  * BiPredicate\<T, U\>, BiFunction\<T, U, R\> 등.
  * BiFunction은 기본 타입을 반환할 때 똑같이 접두어에 ``ToResult``를 사용한다.
    * ToIntBiFunction\<T, U\> 등.
  * Consumer는 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형은 다음과 같다.
    * ObjIntConsumer\<T\>, ObjLongConsumer\<T\> 등.
* Supplier는 boolean을 반환할 때 BooleanSupplier를 사용할 수 있다.

<br>

## 5. 타입 검사 및 추론

### 5.1. 타입 검사

컴파일러는 람다가 사용되는 문맥을 통해 람다의 타입을 검사 및 추론한다.

> Filter.java

```java
Collections.sort(list, (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

* ``sort()`` 메서드의 선언부분을 확인한다.
* 두 번째 파라미터가 Comparator\<Apple\>임을 확인함으로써 타깃 타입을 한정한다.
* 해당 함수형 인터페이스가 사용하는 추상 메서드를 확인한다.
  * Apple을 파라미터로 받아 int를 리턴한다.
* ``sort()`` 메서드로 전달되는 람다식이 이러한 요구사항에 부합하는지 검사한다.
* **만약 람다 표현식이 예외를 호출한다면, 추상 메서드의 throws 절에 선언된 예외와 부합해야 한다.**

서로 다른 함수형 인터페이스들이 상호 호환되는 추상 메서드 시그니처를 가지고 있다면 같은 람다 표현식을 사용할 수 있다. Comparator에 사용되는 람다는 BiFunction 및 ToIntBiFunction 등에 사용이 가능하다.

> Ambiguous.java

```java
public void execute(Runnable runnable) {
     runnable.run();  
}

public void execute(Action<T> action) {
    action.act();
}

execute((Action) () -> {});
```

* 함수형 인터페이스를 인자로 받는 메서드는 오버로딩을 조심해야 한다.
  * Runnable과 Action은 모두 함수 디스크립터(void - void)가 같기 때문에 컴파일러에게 특정 타입을 명시해야 한다.

### 5.2. 타입 추론

> Filter.java

```java
Collections.sort(list, (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

* 함수 디스크립터를 통해 타입을 검사하기 때문에, 람다의 적절한 시그니처를 추론할 수 있다.
  * 함께 사용되는 list의 타입 매개변수 등 문맥을 분석한다.
* 람다 표현식의 파라미터 타입은 생략이 가능하다.

### 5.3. 지역 변수 사용

람다 표현식은 외부 스코프에 정의되어 있는 자유 변수들을 사용할 수 있다. 인스턴스 변수나 정적 변수를 사용하는데 제한은 없으나, **지역 변수**는 (사실상) final이어야 한다. 또한 람다는 지역 변수를 수정할 수 없다.

> Port.java

```java
int portNumber = 13;
Runnable r = () -> System.out.println(portNumber);
portNumber = 12; //컴파일 에러 발생
```

* 인스턴스 변수는 힙에 저장되는 반면, 지역 변수는 스택에 저장된다.
* 스레드에서 사용되는 람다가 지역 변수에 직접 접근할 수 있게 된다면 스레드 안전성에 악영향을 끼친다.
  * 해당 람다를 사용하는 스레드는 지역 변수를 할당하고 해제한 스레드 이후에 지역 변수에 접근 시도를 할 수 있다.
* 따라서 Java는 원본이 아닌 복사본에 해당하는 자유 지역 변수에 대해서만 람다가 접근하도록 허용한다.
* 클로저(Closure)는 람다와 유사하지만 함수 인스턴스로서 외부 스코프의 변수를 수정할 수 있다.

<br>

## 6. 메서드 참조

메서드 참조는 기존의 메서드 정의를 재사용하고 람다처럼 전달할 수 있게끔 해준다. 메서드 이름을 직접 참조함으로써 람다보다 간결하고 가독성을 높일 수 있다. 람다와 비슷한 타입 검사를 진행하니 메서드 참조의 시그니처는 문맥의 타입과 일치해야 한다.

> Sortings.java

```java
inventory.sort(comparing(Apple::getWeight));
```

* 정적 메서드 참조.
  * ``str -> Integer.parseInt(str)``을 ``Integer::parseInt``로 치환한다.
* 한정적 인스턴스 메서드 참조.
  * 참조 대상 인스턴스를 특정한다.
  * 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 동일하다.
  * ``Instance then = Instance.now();``
  * ``t -> then.isAfter(t)``를 ``then::isAfter``로 치환한다.
* 비한정적 인스턴스 메서드 참조.
  * 참조 대상 인스턴스를 특정하지 않는다.
  * 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
  * ``str -> str.toLowerCase()``를 ``String::toLowerCase``로 치환한다.
* 클래스 생성자.
  * ``() -> new TreeMap<K,V>``를 ``TreeMap<K,V>::new``로 치환한다.
* 배열 생성자.
  * ``len -> new int[len]``을 ``int[]::new``로 치환한다.

<br>

## 7. 람다 표현식을 구성하는 유용한 메서드들

인터페이스가 제공하는 디폴트 메서드들을 통해 람다 표현식을 조금 더 유연하게 구성할 수 있다.

### 7.1. Comparator

> Comparator.java

```java
Comparator <Apple> c = Comparator.comparing(Apple::getWeight);

inventory.sort(Comparator.comparing(Apple::getWeight)
            .reversed()
            .thenComparing(Apple::getCountry));
```

* Function을 비교 기준으로 받아 Comparator 객체를 반환하는 ``comparing()`` 메서드가 있다.
* 역순으로 비교하는 기능 또한 제공한다.
* ``thenComparing()`` 메서드 체이닝을 통해 다중 조건으로 비교 정렬할 수 있다.

### 7.2. Predicate

> Predicate.java

```java
Predicate<Apple> notRedApple = redApple.negate();

Predicate<Apple> redAndHeavyAndAppleOrGreen = redApple
    .and(apple -> apple.getWeight() > 150)
    .or(apple -> Green.equals(apple.getColor()));
```

* Predicate 인터페이스 또한 and, or, not 등의 조건을 메서드 체이닝으로 활용할 수 있다.
* 우선순위는 왼쪽에서 오른쪽 방향순이다.
  * ``a.or(b).and(c)``는 ``(a || b) && c``이다.

### 7.3. Function

> AndThen.java

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1); //4
```

> Compose.java

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g);
int result = h.apply(1); //3
```

* ``andThen()``이 g(f(x))라면 ``compose()``는 f(g(x))를 의미한다.

> LetterChecker.java

```java
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline = addHeader.andThen(Letter::checkSpelling).andThen(Letter::addFooter);
```

* 여러 유틸리티 메서드들을 조합하여 다양한 파이프라인을 생성할 수 있다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
* Effective Java 3/E(Joshua Bloch 저)
