---
title: "[Modern Java in Action] 7장. 병렬 데이터 처리와 성능"
excerpt: "병렬화를 통해 모든 멀티코어 프로세서가 각각의 서브 파트 작업을 처리하도록 할당할 수 있다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-01
last_modified_at: 2021-03-01
---

## 1. 병렬 스트림

``parallel()``을 호출하여 컬렉션의 스트림을 병렬 스트림으로 변환할 수 있다. 병렬 스트림은 내부적으로 스트림 원소들을 여러 개의 서브 파트로 분리하고, 각각의 서브 파트 처리 작업을 서로 다른 스레드들에게 위임한다. 모든 멀티코어 프로세서가 각각의 서브 파트 작업들을 처리하도록 할당(작업 부하 분산)할 수 있다.

병렬 스트림은 내부적으로 사용자의 프로세서 코어 개수만큼의 스레드를 가진 ForkJoinPool을 사용한다. 사용자의 프로세서 코어는 ``Runtime.getRuntime().availableProcessors()``를 통해 얻게 된다. ``System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12")``와 같이 전역 셋팅을 변경시키면 코드의 모든 병렬 스트림들에 영향을 줄 수 있다. 그러나 단일 병렬 스트림에는 특정한 값을 명시할 수 없으며, 특별한 이유가 있지 않는 이상 스레드 개수를 변경하지 않고 기본 값을 사용하는 것이 낫다.

> ParallelStream.java

```java
public static long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
            .limit(n)
            .parallel()
            .filter(...)
            .sequential()
            .map(...)
            .parallel()
            .reduce(...);
}
```

* ``parallel()`` 메서드 호출은 순차 스트림 자체를 직접적으로 변환시키는 것이 아닌, 내부적으로 boolean 플래그를 셋팅하여 ``parallel()`` 호출 이후의 작업들을 병렬로 처리함을 알리는 것이다.
* 병렬 스트림은 ``sequential()`` 호출을 통해 순차 스트림으로 변환할 수 있다.
* 그러나 가장 마지막에 호출된 ``parallel()`` 혹은 ``sequential()``이 파이프라인에 전역적인 영향을 끼치게 된다.
  * 위 예제의 파이프라인은 결국 병렬 스트림이 적용된다.

### 1.1. 벤치마크

JMH 등의 라이브러리를 사용하면 Java API의 성능을 애너테이션 기반으로 간단하게 측정할 수 있다. JVM에서 동작하는 프로그램은 GC에 의한 오버헤드 등 고려해야 할 변수가 많아 API의 성능을 정밀하게 벤치마킹하는 것이 다소 어렵다.

### 1.2. 주의사항

> ParallelStream.java

```java
public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1).limit(n).parallel().reduce(Long::sum).get();
}
```

* 잘못된 병렬 스트림 사용은 일반 반복문 사용 및 순차 스트림 사용보다 성능이 나쁘다.
* ``iterate()`` 메서드는 박싱된 래핑 객체를 반환하기 때문에 연산 중 오토박싱으로 인한 오버헤드가 발생한다.
* ``iterate()`` 메서드로 생성된 스트림은 독립된 서브 파트들로 분할되기 어려운 구조이다.
  * 스트림을 생성하는데 사용되는 입력값은 이전의 결과값에 의존한다.
  * 리듀싱 연산을 시작할 때 모든 숫자 리스트들이 준비되지 않아, 리듀싱 연산이 사실상 순차적인 구조다.
  * 병렬 스트림을 이용하면 여러 스레드들이 순차 연산을 수행하기 때문에 싱글 스레드의 순차 연산보다 느리다.
    * 여러 스레드들에게 작업을 위임하는 불필요한 오버헤드가 발생하기 때문이다.

> ParallelStream.java

```java
public static long parallelRangedSum(long n) {
    return LongStream.rangeClosed(1, n).parallel().reduce(Long::sum).getAsLong();
}
```

* 불필요한 오토박싱을 피할 수 있는 기본 자료형 특화 스트림을 사용한다.
* 특히 ``rangeClosed()``가 생성하는 특정 범위의 숫자들은 서로 독립된 서브 파트들로 분리되기 쉬운 구조이다.
* 병렬 처리를 하지 않고 오토박싱을 피할 수 있는 적절한 **자료 구조**를 사용하는 것만으로도 성능이 크게 개선된다.
* 병렬 처리 작업은 비용이 수반된다.
  * 스트림을 재귀적으로 분리하고, 각각의 스레드에게 각 서브 파트들에 대한 리듀싱 연산 작업을 위임하고, 마지막에 여러 결과들을 하나로 병합하는 과정이 발생한다.
  * 여러 프로세서 코어들간의 데이터 교환 비용은 생각보다 비싸다.

> ParallelStream.java

```java
public static long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
    return accumulator.total;
}

public static class Accumulator {

    private long total = 0;

    public void add(long value) {
        total += value;
    }
}
```

* 병렬 스트림을 사용할 때 가변 상태를 가지는 알고리즘을 사용하는 경우 동시성 문제가 발생한다.
  * 동시성 문제를 해결하기 위해 동기화 작업을 진행하면 병렬 처리 작업의 이점이 사라지게 된다.
* 여러 스레드들이 동시에 Accumulator 객체에 접근하여 값을 변경시키기 때문에 사이드 이펙트가 발생한다.
    * ``add()`` 메서드는 원자적 실행 단위의 연산이 아니다.

병렬 스트림을 효율적으로 사용하고자 한다면 다음 사항들을 고려해본다.

* 벤치마크 등을 통해 성능을 다각도로 비교 측정해보고 병렬 스트림 사용을 고려한다.
* 불필요한 오토박싱 오버헤드만 줄여도 성능이 향상된다.
* 일부 Stream 연산들은 병렬 스트림일 경우 성능이 나쁘다.
  * ``limit()``과 ``findFirst()``는 원소들의 순서에 의존하기 때문에 병렬 작업시 비용이 비싸다.
  * 원소의 순서가 중요하지 않다면 ``findAny()`` 및 ``unordered()`` 등을 통해 병렬 작업 성능을 높일 수 있다.
* 파이프라인 연산 작업의 전체 비용을 고려하라.
  * N개의 스트림 * 각 스트림 처리 비용 Q 수식에서 Q의 값이 높다면 병렬 처리를 통해 성능을 향상시킬 가능성이 존재한다.
  * 반면 N이 너무 작으면 병렬화를 통한 이점이 없다.
* 스트림의 소스가 병렬 작업시 여러 서브 파트들로 쉽게 분리될 수 있는 자료 구조인지 고민한다.
  * ArrayList는 원소들을 순회할 필요 없이 균등하게 분리될 수 있지만 LinkedList는 아니다.
  * 기본 자료형 특화 스트림은 쉽게 분리될 수 있다.
  * Spliterator 구현을 통해 스트림의 서브 파트 분리 작업을 직접 제어할 수 있다.
  * HashSet과 TreeSet은 Decomposability가 나쁘지 않은 편이다.
* 스트림 및 파이프라인의 중간 연산 특징에 따라 스트림 분리 작업의 성능이 천차만별이다.
  * 사이즈가 고정된 스트림은 쉽게 균등한 크기의 여러 서브 파트들로 분리될 수 있으나, 필터링 연산은 예측 불가능한 개수의 원소를 반환하기 때문에 스트림의 크기를 알 수 없다.
* 종단 연산의 병합 작업에 대한 비용을 고려하라.
  * ``combiner()``의 병합 비용이 너무 크면 병렬 작업 처리를 통한 성능의 이점을 상쇄하게 된다.

<br>

## 2. Fork-Join Framework

fork-join 프레임워크는 병렬 처리가 가능한 작업을 서브 작업들로 재귀적으로 분리한 다음 서브 작업들의 연산 결과를 병합하여 최종 결과를 생산한다. ExecutorService 인터페이스 구현을 통해 ForkJoinPool 스레드 풀의 작업 스레드들에게 서브 작업들을 분배한다. 분할 정복 알고리즘을 사용한다.

> ForkJoinSumCalculator.java

```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {

    public static final long THRESHOLD = 10_000;

    private final long[] numbers;
    private final int start;
    private final int end;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
        leftTask.fork();
        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}
```

* RecursiveTask를 구현할 때 타입 매개변수 R은 연산의 리턴 타입이다.
  * 리턴 타입이 없는 경우 RecursiveAction이다.
* ``compute()`` 메서드는 작업을 분리하고 수행 및 병합하는 핵심 알고리즘이 포함되어 있다.
  * 작업을 더 나눌지 말지에 대한 명확한 기준은 존재하지 않는다.

> ForkJoinSumCalculator.java

```java
private static final ForkJoinPool FORK_JOIN_POOL = new ForkJoinPool();

public static long forkJoinSum(long n) {
    long[] numbers = LongStream.rangeClosed(1, n).toArray();
    ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
    return FORK_JOIN_POOL.invoke(task);
}
```

* RecursiveTask의 상위 클래스인 ForkJoinTask를 생성하고 ForkJoinPool에 넘겨준다.
* 스레드 풀은 한 개만 존재해도 되기 때문에 싱글톤으로 관리한다.
  * ForkJoinPool의 기본 생성자는 JVM에서 사용 가능한 모든 프로세서들을 스레드 풀에서 사용한다.
  * ``Runtime.availableProcessors()``를 통해 사용할 스레드의 개수를 결정한다.
    * 하이퍼스레등 등을 통해 생성된 가상 코어들을 포함한 모든 사용 가능한 코어들의 개수를 반환한다.

### 2.1. 주의사항

* 특정 작업에 대해 ``join()``을 호출하면 해당 작업의 결과가 준비될 때 까지 호출된 쪽을 블락한다.
  * 따라서 양 쪽의 서브 작업들이 시작된 이후에 호출해야 하며, 그렇지 않으면 서브 작업들은 다른 서브 작업들이 완료될 때 까지 실행되지 못하고 기다리게 된다.
* ForkJoinPool의 ``invoke()`` 메서드는 RecursiveTask 내부에서 사용해서는 안 된다.
  * 대신 ``compute()``나 ``fork()``를 직접 사용한다.
  * 병렬 작업을 실행하려는 순차 코드만이 ``invoke()``를 사용한다.
* 서브 작업에서 ``fork()``를 호출하는 것은 ForkJoinPool에서 작업을 스케쥴링하는 것과 같다.
  * 왼쪽과 오른쪽 모두 ``fork()``를 호출하는 것 보다, 한 쪽은 ``compute()``를 직접 호출하는 것이 더 효과적이다.
    * 두 서브 작업들 중 하나를 처리하는데 같은 스레드를 재사용할 수 있으며, 스레드 풀에서 불필요한 추가 서브 작업 할당으로 인해 발생되는 오버헤드를 피할 수 있다.
* 스택 추적으로도 디버깅하기가 다소 어렵다.
  * ``compute()``는 개념적인 호출자 쪽(``fork()``를 호출한 영역)이 아닌 다른 스레드에서 표시되기 때문이다.
* 각 서브 작업들의 실행 시간은 새로운 서브 작업을 fork하는데 걸리는 시간보다 길어야 한다.
* 제대로된 성능 비교를 위해서는 벤치마킹 검사 전에 여러 번 테스트 수행 등의 **워밍 업**이 필요하다.
  * 코드를 몇 번 실행해야 JIT 컴파일러에 의해 최적화되기 때문이다.

### 2.2. 작업 훔치기

위 예제 코드는 n이 클 경우 천 개가 넘는 서브 작업들을 fork한다. CPU 코어의 개수는 한정적이라서 자원을 낭비하는 것 같지만, 일반적으로 많은 수의 서브 작업들을 fork하는 것이 더 큰 효율을 이끌어낼 수 있다. 병렬 처리의 목적은 작업의 부하를 모든 CPU 코어에게 동등하게 분산시킴으로써 처리 속도를 높이는 것이다. 그러나 여러 원인들로 인해 서브 작업들마다 처리 소요 시간이 상이하기 때문에, 만약 작업을 다 끝낸 CPU가 유휴 상태에 돌입하면 모든 CPU 코어의 능력을 100%로 활용할 수 없다.

작업 훔치기란 작업을 다 끝낸 스레드가 유휴 상태에 돌입하는 대신 다른 스레드의 작업을 가지고 와서 처리해주는 알고리즘이다.

* 각 스레드는 자신에게 할당된 작업에 대한 양방향 연결 큐를 가지고 있다.
* 할당된 작업을 전부 수행해 큐가 비워진 스레드는 다른 스레드의 작업 큐의 꼬리에서 작업을 훔쳐온다.
  * 모든 서브 작업들이 완료될 때 까지 이 과정이 반복된다.
* 이러한 작업 훔치기 알고리즘은 스레드 풀의 작업 스레드들 사이에서의 서브 작업을 재배분하고 로드 밸런싱하는데 사용된다.
  * 적은 수의 큰 작업들보다 많은 수의 작은 작업들을 사용하는 것이 스레드간의 작업 부하를 비슷한 수준으로 유지할 수 있다.

<br>

## 3. Spliterator

Spliterator는 병렬 작업시 스트림 원소들을 순회할 때 사용되는 Iterator다. Java 8 부터 컬렉션 프레임워크에 새롭게 추가된 인터페이스다. 병렬 스트림이 순회하는 데이터를 어떤 기준으로 분리할지 정의한다.

> Spliterator.java

```java
public interface Spliterator<T> {

    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

* ``tryAdvance()`` 메서드는 일반 Iterator처럼 순차적으로 원소들을 소비할 때 사용된다.
  * 더 순회할 원소가 남아있는 경우 true를 반환한다.
* ``trySplit()`` 메서드는 스트림 원소들 중 일부를 메서드를 통해 반환되는 두 번째 Spliterator에게 파티셔닝한다.
* ``estimateSize()`` 메서드는 아직 더 순회할 수 있는(남아있는) 원소들의 개수 추정치를 반환한다.
  * ``characteristics()`` 메서드에 SIZED 플래그가 세워진 경우 사이즈를 알 수 있어 정확한 값을 반환한다.

![image](https://user-images.githubusercontent.com/56240505/109487147-2ef7bb80-7ac7-11eb-8ab7-24abb2135d91.png)

* Spliterator들은 계속 ``trySplit()``을 통해 새로운 Spliterator를 재귀적으로 생성한다.
* 더 이상 나눠질 수 없는 경우 null을 반환하여 자료 구조에게 알린다.
* 모든 Spliterator가 null을 반환할 때 재귀적인 생성 작업이 중단된다.
* 이러한 분리 작업은 Spliterator의 ``characteristics()`` 메서드에 영향을 받는다.
  * Spliterator 사용 및 최적화에 대한 옵션을 지정할 수 있다.
  * ORDERED, DISTINCT, SORTED, SIZED, NON-NULL, IMMUTABLE, CONCURRENT, SUBSIZED 등.

### 3.1. 예제

> WordCounter.java

```java
public class WordCounter {

    private final int counter;
    private final boolean lastSpace;

    public WordCounter(int counter, boolean lastSpace) {
        this.counter = counter;
        this.lastSpace = lastSpace;
    }

    public WordCounter accumulate(Character c) {
        if (Character.isWhitespace(c)) {
            return lastSpace ? this : new WordCounter(counter, true);
        }
        else {
            return lastSpace ? new WordCounter(counter + 1, false) : this;
        }
    }

    public WordCounter combine(WordCounter wordCounter) {
        return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
    }

    public int getCounter() {
        return counter;
    }
}
```

* String에서 공백으로 구분된 단어의 개수를 계산하는 기능을 병렬 스트림으로 구현해본다.
* Java는 튜플이 존재하지 않으며 병렬 처리시 가변 상태가 아닌 불변을 유지해야 하기 때문에 매번 새로운 WordCounter 객체를 생성해준다.

> WordCounter.java

```java
Stream<Character> stream; //String 원소들의 스트림
WordCounter wordCounter = stream.reduce(new WordCounter(0, true), WordCounter::accumulate, WordCounter::combine);
int wordCounts = wordCounter.getCounter();
```

* 순차적인 경우 잘 동작하지만, 병렬 스트림을 사용하면 결과가 이상해진다.
  * 병렬 처리시 String이 임의의 지점에서 분리되며, 어떤 경우 단어가 두 개로 분리되고 두 개로 카운팅되기 때문이다.

> WordCounterSpliterator.java

```java
public class WordCounterSpliterator implements Spliterator<Character> {

     private final String string;
     private int currentChar = 0;

     public WordCounterSpliterator(String string) {
         this.string = string;
     }

     @Override
     public boolean tryAdvance(Consumer<? super Character> action) {
         action.accept(string.charAt(currentChar++));
         return currentChar < string.length();
     }

     @Override
     public Spliterator<Character> trySplit() {
         int currentSize = string.length() - currentChar;
         if (currentSize < 10) {
             return null;
         }
         for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
             if (Character.isWhitespace(string.charAt(splitPos))) {
                 Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentChar, splitPos));
                 currentChar = splitPos;
                 return spliterator;
             }
         }
         return null;
     }

     @Override
     public long estimateSize() {
         return string.length() - currentChar;
     }

     @Override
     public int characteristics() {
         return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
     }
}
```

> Main.java

```java
Spliterator<Character> spliterator = new WordCounterSpliterator(SENTENCE);
Stream<Character> stream = StreamSupport.stream(spliterator, true);
WordCounter wordCounter = stream.reduce(new WordCounter(0, true), WordCounter::accumulate, WordCounter::combine);
int wordCounts = wordCounter.getCounter();

```

* ``StreamSupport.stream()`` 메서드의 두 번째 boolean 인자를 통해 병렬 스트림임을 명시한다.

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
