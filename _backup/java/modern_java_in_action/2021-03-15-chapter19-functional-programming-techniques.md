---
title: "[Modern Java in Action] 19장. 함수형 프로그래밍 기법"
excerpt: "고차원 함수란 하나 이상의 함수를 인자로 받거나, 함수를 결과로 반환하는 함수를 의미한다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-15
last_modified_at: 2021-03-15
---

## 1. 함수

일급 함수란 일반값처럼 취급할 수 있는 함수를 의미한다. Java에서 람다 표현식 혹은 메서드 참조 등을 통해 함수를 메서드의 인수나 결과값으로 반환할 수 있으며 컬렉션에 저장도 가능하다.

### 1.1. 고차원 함수

고차원 함수란 하나 이상의 함수를 인자로 받거나, 함수를 결과로 반환하는 함수를 의미한다. ``Comparator.comparing()``이나 ``Function.andThen()`` 등이 고차원 함수로 분류된다.

* 스트림 연산이나 고차원 함수에 전달하는 함수는 부작용이 없어야 한다.
  * 부작용이 존재하는 함수는 부정확한 결과가 발생하거나 스레드 경쟁 상태로 인해 예상하지 못한 결과가 발생할 수 있다.
* 고차원 함수를 구현할 때 인수가 부작용을 포함할 가능성을 염두에 두어야 한다.
  * 함수를 인수로 받아 사용하면 작업 수행 및 프로그램 상태 변화 등을 정확하게 예측하기 어렵다.
  * 인수로 전달된 함수가 어떤 부작용을 포함하게 될지 문서화하는 것이 좋다.

### 1.2. 커링

커링(Currying)이란 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다. g라는 함수 역시 하나의 인수를 받는 함수를 반환한다. 함수를 모듈화하고 코드를 재사용하는데 도움을 준다.

> Converter.java

```java
static double converter(double x, double f, double b) {
    return x * f + b;
}
```

* 특정 단위 값 x를 다른 단위로 변환하는 경우, 변환 요소와 각종 기준치가 필요하다.
  * 위 메서드는 미터, 섭씨, 화씨, 길이 등 다양한 단위를 변환할 수 있으나 오타가 나기 쉽고 가독성이 나쁘다.

> Converter.java

```java
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}

public static void main(String[] args) {
  DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
  DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);
  DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);

  double gbp = convertUSDtoGBP.applyAsDouble(1000);
}
```

* 인수 두 개를 대입하면 인수 한 개를 갖는 변환 함수를 반환하는 팩토리를 정의한다.
* ``f(x, y) = (g(x))(y)`` 커링이 성립한다.

<br>

## 2. 영속 자료구조

함수형 프로그래밍에서는 함수형 자료구조, 불변 자료구조를 보통 영속 자료구조라고 칭한다. DB에서 프로그램 종료 후에도 남아있음을 의미하는 영속과는 다른 의미이다. 함수형 메서드는 전역 자료구조나 인수로 전달된 자료구조를 업데이트할 수 없다. 자료구조를 변경한다면 같은 메서드를 두 번 호출했을 때 결과가 달라져 참조 투명성을 위배하게 된다.

### 2.1. 파괴적인 갱신과 함수형

> TrainJourney.java

```java
public class TrainJourney {

    public int price;
    public TrainJourney onward;

    public TrainJourney(int p, TrainJourney t) {
      price = p;
      onward = t;
    }
}

static TrainJourney link(TrainJourney a, TrainJourney b) {
    if (a == null) return b;
    TrainJourney t = a;
    while (t.onward != null) {
        t = t.onward;
    }
    t.onward = b;
    return a;
}
```

* LinkedList 형태의 TrainJourney를 연결할 때 자료구조를 가변시키는 경우 부작용이 발생한다.
  * a가 X에서 Y로 가는 경로이고, b가 Y에서 Z로 가는 경로일 때 위 연결 메서드는 a의 상태를 X에서 Z로 가는 경로로 변경한다.
  * X-Y 경로의 a 변수에 의존하던 코드들이 깨질 수 있다.

> TrainJourney.java

```java
static TrainJourney append(TrainJourney a, TrainJourney b) {
    return a == null ? b : new TrainJourney(a.price, append(a.onward, b));
}
```

* 계산 결과를 표현하는 자료구조를 새롭게 생성함으로써 부작용을 없앨 수 있다.
  * 추가적인 객체 생성 비용이 들지라도 유지보수 측면에서 낫다.
* LinkedList 경로에 위치한 모든 원소들을 새로 생성하는 것은 아니다.
  * a 변수가 참조하고 있는 경로들에 한해서만 새로 객체를 생성하며, a 변수의 마지막 원소는 b를 참조하게 된다.

> Tree.java

```java
public class Tree {

    private String key;
    private int val;
    private Tree left, right;

    public Tree(String k, int v, Tree l, Tree r) {
        key = k; val = v; left = l; right = r;
    }

    public static Tree update(String k, int newval, Tree t) {
        if (t == null)
            t = new Tree(k, newval, null, null);
        else if (k.equals(t.key))
            t.val = newval;
        else if (k.compareTo(t.key) < 0)
            t.left = update(k, newval, t.left);
        else
            t.right = update(k, newval, t.right);
        return t;
    }
}
```

* Tree 자료구조를 업데이트하는 위 메서드는 결과적으로 가변 상태를 가지기 때문에 부작용이 존재한다.
  * TreeMap 사용자들에게 가변 상태가 노출된다.

> Tree.java

```java
public static Tree fupdate(String k, int newval, Tree t) {
    return (t == null) ? new Tree(k, newval, null, null) :
        k.equals(t.key) ? new Tree(k, newval, t.left, t.right) :
        k.compareTo(t.key) < 0 ?
        new Tree(t.key, t.val, fupdate(k,newval, t.left), t.right) :
        new Tree(t.key, t.val, t.left, fupdate(k,newval, t.right));
}
```

* ``fupdate()`` 메서드는 기존의 자료구조를 오염시키지 않는 순수 함수이다.
* 마찬가지로 업데이트를 진행할 때마다 기존 트리에 속한 모든 원소 객체를 새로 생성하는 것이 아니다.
  * 살릴 수 있는 객체 참조들은 유지하고, 변경이 발생하는 원소 객체들에 한해서만 새로 생성한다.
  * Tree의 균형이 잘 맞는 경우 일반적으로 ``fupdate()`` 방식은 비싸지 않다.
* 이러한 영속 자료구조의 전제조건은 결과로 반환되는 자료구조를 변경하지 않는 것이다.
  * 필드를 final로 설정한다.
    * 다만 필드가 참조하고 있는 객체의 변화까지 보호할 수 있다는 보장은 없다.
    * 필드의 참조 객체 또한 불변이어야 한다.
  * 클라이언트가 자료구조를 업데이트할 때 API를 거치게 하며, API는 최신 버전의 자료구조를 반환하도록 한다.

<br>

## 3. 스트림과 게으른 평가

게으른 평가란 특정한 작업이 일어나야 하는 상황에서만 스트림을 평가하는 것을 의미한다. Java 8의 스트림에 중간 연산들이 주어지더라도 곧바로 수행되는 것이 아니라, 종단 연산을 적용하는 시점에 실제 연산이 이루어진다. 스트림은 원소들을 단 한 번만 순회하기 때문이다.

> PrimeNumbers.java

```java
static int head(IntStream numbers) {
    return numbers.findFirst().getAsInt();
}

static IntStream tail(IntStream numbers) {
    return numbers.skip(1);
}

static IntStream primes(IntStream numbers) {
    int head = head(numbers);
    return IntStream.concat(IntStream.of(head),
        primes(tail(numbers).filter(n -> n % head != 0)));
}
```

* 스트림은 단 한 번만 소비할 수 있기 때문에 재귀적인 호출이 불가능하다.
  * 위 소수 찾기 예제는 종단 연산을 두 번 호출하기 때문에 예외가 발생한다.
* 설사 두 번 소비할 수 있더라도 ``concat()``의 두 번째 인자가 즉시 평가되기 때문에 무한 재귀에 빠지게 된다.

### 3.1. LazyList

일반적인 LinkedList는 모든 원소들이 메모리에 존재하지만, LazyList는 요청이 발생했을 때 함수에 의해 원소가 추가된다.

> LazyList.java

```java
public class LazyList<T> implements MyList<T>{

    final T head;
    final Supplier<MyList<T>> tail;

    public LazyList(T head, Supplier<MyList<T>> tail) {
        this.head = head;
        this.tail = tail;
    }

    public T head() {
        return head;
    }

    public MyList<T> tail() {
        return tail.get();
    }

    public boolean isEmpty() {
        return false;
    }

    public static LazyList<Integer> from(int n) {
        return new LazyList<Integer>(n, () -> from(n+1));
    }
}
```

> Main.java

```java
LazyList<Integer> numbers = from(2);
int two = numbers.head();
int three = numbers.tail().head();
int four = numbers.tail().tail().head();
```

* List의 원소들을 처음부터 모두 초기화해서 한 번에 메모리에 저장하는 것이 아닌, 요청이 있을 때 마다 게으르게 추가하는 방식을 적용한다.
  * 람다 표현식(함수)은 인자로 주어질 때 즉시 평가되지 않고, 실제로 호출되는 시점에 게으르게 평가된다.

> LazyList.java

```java
public static MyList<Integer> primes(MyList<Integer> numbers) {
    return new LazyList<>(numbers.head(),
        () -> primes(numbers.tail().filter(n -> n % numbers.head() != 0)));
}

public MyList<T> filter(Predicate<T> p) {
    return isEmpty() ? this :
        p.test(head()) ? new LazyList<>(head(), () -> tail().filter(p)) : tail().filter(p);
}
```

> Main.java

```java
LazyList<Integer> numbers = from(2);
int two = primes(numbers).head();
int three = primes(numbers).tail().head();
int five = primes(numbers).tail().tail().head();
```

* 자료구조를 10% 미만으로 탐색하는 것이 아니라면 Supplier 등을 통한 게으른 평가의 오버헤드가 장점을 상쇄한다.
  * 전통적인 방식의 자료구조 사용과의 효율성 측면 등을 비교해서 프로그램 제작에 더 도움이 되는 방법을 선택해야 한다.

<br>

## 4. 패턴 매칭

패턴 매칭이란 함수형 프로그래밍을 구분하는 또 다른 특징으로서, 자료형을 언랩하는 함수형 기능이다. 이 때 패턴은 정규표현식의 패턴이 아니다. Java에서는 지원하지 않지만 Scala나 Kotlin 등 모던 언어들은 지원한다.

> Scala

```scala
def simplifyExpression(expr: Expr): Expr = expr match {
    case BinOp("+", e, Number(0)) => e + 0
    case BinOp("*", e, Number(1)) => e * 1
    case BinOp("/", e, Number(1)) => e / 1
    case _ => expr
}
```

* Scala는 표현 지향적이라서 트리와 같은 자료구조를 조작할 때 매우 간편하다.
  * ``Expression match { case Pattern => Expression ... }``
* 패턴 매칭이 지원되는 언어는 장황한 조건문의 체이닝을 획기적으로 줄일 수 있다.

> BinOp.java

```java
Expr simplifyExpression(Expr expr) {
    if (expr instanceof BinOp
        && ((BinOp)expr).opname.equals("+"))
        && ((BinOp)expr).right instanceof Number
        && ... // it's all getting very clumsy
        && ... ) {
          return (Binop) expr.left;
    }
}
```

* Expr 하위 클래스로 Number와 BinOp가 있을 때, Java가 Scala와 같은 기능의 메서드를 만드려면 코드가 매우 장황해진다.
* Java는 선언 지향이며 switch-case 문법으로는 기본 자료형, String, 열거형, 일부 래퍼 클래스 밖에 사용할 수 없어 제약이 크다.

### 4.1. 방문자 패턴

> BinOp.java

```java
public class BinOp extends Expr {

    //...

    public Expr accept(SimplifyExprVisitor v) {
        return v.visit(this);
    }
}
```

> SimplifyExprVisitor.java

```java
public class SimplifyExprVisitor {

    //...

    public Expr visit(BinOp e){
        if ("+".equals(e.opname) && e.right instanceof Number && ...) {
            return e.left;
        }
        return e;
    }
}
```

* 데이터 타입을 언래핑하는 방법으로는 방문자 패턴이 존재한다.
* 특정한 데이터 타입을 방문하는 알고리즘을 캡슐화하는 개별 클래스를 선언하고, 인자로 받은 데이터 타입 인스턴스의 멤버들에 접근한다.

### 4.2. 패턴 매칭 흉내내기

> PatternMatching.java

```java
static <T> T patternMatchExpr(Expr e, TriFunction<String, Expr, Expr, T> binopcase, Function<Integer, T> numcase, Supplier<T> defaultcase) {
    if (e instanceof BinOp) {
        return binopcase.apply(((BinOp) e).opname, ((BinOp) e).left, ((BinOp) e).right);
    }
    else if (e instanceof Number) {
        return numcase.apply(((Number) e).val);
    }
    else {
        return defaultcase.get();
    }
}  
```

> Main.java

```java
TriFunction<String, Expr, Expr, Expr> binopcase = (opname, left, right) -> {
    if ("+".equals(opname)) {
        if (left instanceof Number && ((Number) left).val == 0) {
            return right;
        }
        if (right instanceof Number && ((Number) right).val == 0) {
            return left;
        }
    }
    if ("*".equals(opname)) {
        if (left instanceof Number && ((Number) left).val == 1) {
            return right;
        }
        if (right instanceof Number && ((Number) right).val == 1) {
            return left;
        }
    }
    return new BinOp(opname, left, right);
};
Function<Integer, Expr> numcase = val -> new Number(val);
Supplier<Expr> defaultcase = () -> new Number(0);

Expr e = new BinOp("+", new Number(5), new Number(0));
Expr match = patternMatchExpr(e, binopcase, numcase, defaultcase);
System.out.println(match); //5
```

* Scala의 패턴 매칭은 멀티 레벨이지만, Java에서 람다를 통해 흉내낼 수 있는 패턴 매칭은 싱글 레벨이다.

<br>

## 5. 기타

> Caching.java

```java
private final Map<Range,Integer> numberOfNodes = new HashMap<>();

Integer computeNumberOfNodesUsingCache(Range range) {
    return numberOfNodes.computeIfAbsent(range, this::computeNumberOfNodes);
}
```

* 반환 객체의 생성 비용이 비싼 경우 캐싱을 통해 비용을 절감할 수 있다.
* 캐싱 메서드 자체는 HashMap을 가변시키지만 상태를 외부로 노출하지 않아 참조 투명성을 위배하지 않는다.
  * 그러나 공유 가변 상태를 가지고 동기화가 되지 않아 스레드 안전하지 않다.
  * ConcurrentHashMap을 사용하면 멀티코어 시스템에서 메서드를 병렬 요청할 때 기대 성능을 발휘하기 어렵다.
    * Map에 특정 Range가 있는지 확인하는 작업과 Map에 엔트리를 추가하는 작업 사이에서 경쟁 상황이 발생한다.
    * 여러 프로세서가 같은 값을 계산하고 Map에 추가할 수 있다.
* 함수형 프로그래밍으로 작성된 코드는 대부분의 경우 함수형 메서드가 동기화 되었는지 걱정할 필요가 없다.
  * 공유 가변 상태가 없기 때문이다.

> Transparency.java

```java
Tree tree = new Tree();
Tree t2 = fupdate("Will", 26, tree);
Tree t3 = fupdate("Will", 26, tree);
System.out.println(t2 == t3); //false
```

* 참조 투명성이란 함수가 같은 인수에 대해 항상 같은 결과를 반환해야 한다는 제약이다.
* Tree를 업데이트하는 메서드를 두 번 호출하면 서로 다른 레퍼런스를 반환한다.
  * ``t2 == t3``는 거짓이다.
* 하지만 ``fupdate()``는 변경되지 않는 영속 자료구조를 사용하고 있으며, t2와 t3간 논리적인 차이가 존재하지 않는다.
* 따라서 함수형 프로그래밍에서는 **일반적으로** ``==`` 레퍼런스 비교 대신 ``equals()``를 통해 구조화된 값들을 비교한다.
  * 데이터가 수정되지 않았기 때문에 현재 모델에서 ``fupdate()`` 메서드는 참조 투명성을 위배하지 않는다.

> Combinator.java

```java
static <A,B,C> Function<A,C> compose(Function<B,C> g, Function<A,B> f) {
    return x -> g.apply(f.apply(x));
}

static <A> Function<A,A> repeat(int n, Function<A,A> f) {
    return n==0 ? x -> x : compose(f, repeat(n - 1, f));
}

public static void main(String[] args) {
    System.out.println(repeat(3, (Integer x) -> 2*x).apply(10)); //80
}
```

* 콤비네이터는 둘 이상의 함수나 자료구조를 조합하는 함수형 개념이다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
