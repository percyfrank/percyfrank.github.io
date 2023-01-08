---
title: "[Modern Java in Action] 9장. 리팩토링, 테스팅, 디버깅"
excerpt: "가독성이 좋은 코드는 작성자가 아닌 타인이 쉽게 코드를 이해하고 유지보수하는데 기여한다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-08
last_modified_at: 2021-03-08
---

## 1. 리팩토링

가독성이 좋은 코드는 코드 작성자가 아닌 타인이 쉽게 코드를 이해하고 유지보수하는데 기여한다. 장황한 익명 클래스 관용구를 람다 표현식으로 축약할 수 있으며, 람다 표현식은 또한 메서드 참조로 축약이 가능하다. 또한 컬렉션 데이터를 처리하던 기존의 명령형 프로그래밍(How) 방식은 스트림(What)을 통해 간단하게 축약될 수 있다.

> Logger.java

```java
if (logger.isLoggable(Level.FINER)) {
    logger.finer("Problem: " + generateDiagnostic());
}
```

* 위 코드는 다음과 같은 문제가 존재한다.
  * 로거의 상태가 클라이언트 코드에 노출된다.
  * 로거 객체의 상태에 대해 항상 쿼리해야 하는 번거로움이 있다.

> Logger.java

```java
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
```

* 기존의 방식보다는 개선되었으나 로거가 로거를 작성하지 못하는 경우에도 로그 메시지가 항상 평가되어야 한다.
* 람다 표현식을 통해 값을 게으르게 지연 생성할 수 있다.

> Logger.java

```java
public void log(Level level, Supplier<String> msgSupplier) {
    if (logger.isLoggable(level)) {
        log(level. msgSupplier.get());
    }
}
```

* ``orElse()``와 ``orElseGet()``의 차이를 생각해보자.

<br>

## 2. 람다를 통한 OOP 디자인 패턴 리팩토링

디자인 패턴이란 소프트웨어 디자인 과정에서 자주 직면할 수 있는 상황들을 해결하기 위한 일종의 청사진(재사용 가능한 코드)이다. 디자인 패턴을 구현하는데 사용되는 번거로운 코드들을 람다 표현식을 통해 간결하게 축약할 수 있다.

### 2.1. 전략 패턴

전략 패턴이란 여러 알고리즘들을 표현하고 런타임 시점에 원하는 알고리즘을 선택할 수 있는 기법이다.

> ValidatorStrategy.java

```java
// old school
Validator v1 = new Validator(new IsNumeric());
System.out.println(v1.validate("aaaa"));
Validator v2 = new Validator(new IsAllLowerCase());
System.out.println(v2.validate("bbbb"));

// with lambdas
Validator v3 = new Validator((String s) -> s.matches("\\d+"));
System.out.println(v3.validate("aaaa"));
Validator v4 = new Validator((String s) -> s.matches("[a-z]+"));
System.out.println(v4.validate("bbbb"));

```

* 전략 인터페이스의 구체 클래스를 별도의 파일로 관리하거나 익명 클래스를 선언할 필요 없이 람다로 간단하게 축약할 수 있다.

### 2.2. 템플릿 메서드 패턴

템플릿 메서드 패턴은 어느 알고리즘의 개괄을 표현하면서 동시에 해당 알고리즘의 일부분을 수정할 수 있는 유연성을 얻고자 할 때 사용하는 기법이다.

> OnlineBanking.java

```java
abstract class OnlineBanking {

    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
      }

    abstract void makeCustomerHappy(Customer c);
}
```

* ``processCustomer()`` 메서드는 온라인 뱅킹 알고리즘에 대한 골격을 제공한다.
* 해당 추상 클래스를 확장하여 고객을 처리하는 알고리즘의 일부분을 수정하여 사용할 수 있다.


> OnlineBanking.java

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}
```

* 람다 표현식을 통해 별도의 추상 메서드 구현 없이 템플릿 메서드 패턴을 구현할 수 있다.

### 2.3. 옵저버 패턴

옵저버 패턴은 특정 이벤트가 발생했을 때 Subject 객체가 다른 Observer 객체 리스트에 자동으로 Notification을 보내는 기법이다. GUI 등에서 버튼이 클릭됐을 때 다른 옵저버들에게 이러한 사실을 알리고 특정 행동을 수행하는 것이 대표적인 예이다.

> Observer.java

```java
public class ObserverMain {

    interface Observer {
        void inform(String tweet);
    }

    interface Subject {
        void registerObserver(Observer o);
        void notifyObservers(String tweet);
    }

    static private class NYTimes implements Observer {

        @Override
        public void inform(String tweet) {
            if (tweet != null && tweet.contains("money")) {
                System.out.println("Breaking news in NY!" + tweet);
            }
        }
    }

    static private class Guardian implements Observer {

        @Override
        public void inform(String tweet) {
            if (tweet != null && tweet.contains("queen")) {
                System.out.println("Yet another news in London... " + tweet);
            }
        }
    }

    static private class LeMonde implements Observer {

        @Override
        public void inform(String tweet) {
            if (tweet != null && tweet.contains("wine")) {
                System.out.println("Today cheese, wine and news! " + tweet);
            }
        }
    }

    static private class Feed implements Subject {

        private final List<Observer> observers = new ArrayList<>();

        @Override
        public void registerObserver(Observer o) {
            observers.add(o);
        }

        @Override
        public void notifyObservers(String tweet) {
            observers.forEach(o -> o.inform(tweet));
        }
    }
}
```

* Feed는 등록된 신문사 Observe 객체들에게 Notification을 보내고, 알맞은 Observer 객체가 이에 반응한다.
* 마찬가지로 Observer 인터페이스는 함수형 인터페이스이기 때문에 Feed에 옵저버를 등록할 때 ``inform()`` 메서드에 상응하는 람다 표현식을 제공해도 된다.
* 다만 옵저버가 복잡하거나 상태를 가지고 있으며 함수형 인터페이스가 아닌 경우 람다 표현식이 아닌 별도의 클래스 파일로 옵저버를 관리해야 한다.

### 2.4. 책임 연쇄 패턴

책임 연쇄 패턴은 객체들을 처리하는 연쇄적인 체이닝을 생성하는 기법이다.

> ChainOfResponsibility.java

```java
public class ChainOfResponsibility {

    private static abstract class ProcessingObject<T> {

        protected ProcessingObject<T> successor;

        public void setSuccessor(ProcessingObject<T> successor) {
            this.successor = successor;
        }

        public T handle(T input) {
            T r = handleWork(input);
            if (successor != null) {
                return successor.handle(r);
            }
            return r;
        }

        abstract protected T handleWork(T input);
    }

    private static class HeaderTextProcessing extends ProcessingObject<String> {

        @Override
        public String handleWork(String text) {
            return "From Raoul, Mario and Alan: " + text;
        }
    }

    private static class SpellCheckerProcessing extends ProcessingObject<String> {

        @Override
        public String handleWork(String text) {
            return text.replaceAll("labda", "lambda");
        }
    }
}
```

* 특정 작업에 대한 다음 연쇄 처리 객체를 등록하고 작업을 위임하는 방식이다.
* 추상 클래스를 구현해야 하는 번거로움이 있다.

> ChainOfResponsibilityMain.java

```java
UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);
String result2 = pipeline.apply("Aren't labdas really sexy?!!");
```

* 람다 표현식을 파이프라인으로 구성해 책임 연쇄 패턴을 구현할 수 있다.

### 2.5. 팩토리 메서드 패턴

팩토리 메서드 패턴은 클라이언트에게 인스턴스화 로직을 노출하지 않은채 객체를 생성하는 방식이다.

> ProductFactory.java

```java
public class ProductFactory {

    public static Product createProduct(String name) {
        switch (name) {
            case "loan":
                return new Loan();
            case "stock":
                return new Stock();
            case "bond":
                return new Bond();
            default:
                throw new RuntimeException("No such product " + name);
        }
    }
}
```

* ``Product p = ProductFactory.createProduct("loan");`` 같이 인스턴스화 로직을 숨기고 객체를 생성할 수 있다.

> ProductFactory.java

```java
public class ProductFactory {
    private static final Map<String, Supplier<Product>> map = new HashMap<>();

    static {
        map.put("loan", Loan::new);
        map.put("stock", Stock::new);
        map.put("bond", Bond::new);
    }

    public static Product createProduct(String name){
        Supplier<Product> p = map.get(name);
        if (p != null) return p.get();
        throw new IllegalArgumentException("No such product " + name);
    }
}
```

* 표준 함수형 인터페이스와 람다 표현식을 통해 팩토리 메서드 패턴을 축약할 수 있다.
* 팩토리 메서드에 인자를 여러 개 받아 객체를 생성하고자 한다면, TriFunction 등 표준 함수형 인터페이스의 변형 버전을 응용하라.

<br>

## 3. 디버깅

프로그램이 메서드를 호출할 때 해당 메서드 호출과 관련된 정보들이 스택 프레임에 저장된다. 개발자는 프로그램이 실패할 때 이러한 스택 프레임에 대한 스택 추적(Stack Trace)을 참조하여 원인을 분석한다. 그러나 람다 표현식은 이름이 없어서 스택 추적에서 문제 원인을 파악하기 어렵다.

메서드 참조 또한 람다 표현식과 마찬가지로 스택 추적에서 이름이 명시되지 않아 읽기 까다롭다. 단 메서드 참조가 사용된 곳이 해당 메서드가 선언된 클래스라면 스택 추적에서 정상적으로 이름이 명시된다.

> Logging.java

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);
numbers.stream()
       .map(x -> x + 17)
       .filter(x -> x % 2 == 0)
       .limit(3)
       .forEach(System.out::println);
```

* ``forEach()``를 사용할 때 전체 스트림을 소비하기 때문에 ``map()``, ``filter()`` 등의 파이프라인 연산이 무엇을 생산해내는지 디버깅하기 어렵다.

> Logging.java

```java
List<Integer> result = consumed from the source.
numbers.stream()
       .peek(x -> System.out.println("from stream: " + x))
       .map(x -> x + 17)
       .peek(x -> System.out.println("after map: " + x))
       .filter(x -> x % 2 == 0)
       .peek(x -> System.out.println("after filter: " + x))
       .limit(3)
       .peek(x -> System.out.println("after limit: " + x))
       .collect(toList());
```

* ``peek()`` 메서드는 스트림을 소비하지 않고 특정한 작업을 수행해준다.
* 이를 통해 파이프라인 연산 전후의 결과를 쉽게 이해할 수 있다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
