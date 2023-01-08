---
title: "[Effective Java] Item 79. 과도한 동기화는 피하라"
excerpt: "교착 상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-11
last_modified_at: 2021-02-11
---

## 1. 과도한 동기화

과도한 동기화는 성능을 떨어뜨리고 교착상태에 빠지는 부작용을 낳는다.

* 응답 불가 및 안전 실패를 피하려면 동기화 메서드나 블록 안에서의 제어를 클라이언트에게 양도해서는 안 된다.
  * 재정의 할 수 있는 메서드나 클라이언트가 넘겨준 함수 객체를 호출해서는 안 된다.
  * 외계인 메서드로 인해 부작용이 발생할 수 있다.

> ObservableSet.java

```java
public class ObservableSet<E> extends ForwardingSet<E> {

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public ObservableSet(Set<E> set) { super(set); }

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```

* 옵저버를 등록하거나 삭제할 때 콜백 인터페이스의 인스턴스를 메서드에 전달한다.

> Test2.java

```java
public class Test2 {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) // 값이 23이면 자신을 구독해지한다.
                    s.removeObserver(this);
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

* 동기화로 인해 ConccurentModificationException이 바생한다.
  * 리스트에서 원소를 제거하려드는 시점이 리스트를 순회하는 도중이기 때문이다.

<br>

## 2. 교착 상태

> Test3.java

```java
public class Test3 {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if (e == 23) {
                    ExecutorService exec =
                            Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get();
                    } catch (ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

* 실행자 서비스를 이용해 다른 스레드에게 구독 해지를 요청한다.
* 백그라운드 스레드는 관찰자를 잠그려고 하지만 메인 스레드가 락을 가지고 있어 얻을 수 없다.
* 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기를 기다리면서 교착 상태에 빠진다.
* 불변식이 깨진 경우 Java의 락은 재진입을 허용하므로 응답 불가(교착 상태)에 빠지지는 않지만, 안전 실패(데이터 훼손)으로 이어질 수 있다.

> ObservableSet.java

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

* 외계인 메서드 호출을 동기화 블록 바깥으로 옮기고, 기존 리스트는 복사해 사용한다.
  * 열린 호출이라고 한다.
  * 락 없이도 안전하게 순회가 가능해진다.
  * 실패 방지 효과와 동시성 효율을 개선해준다.
* 외계인 메서드는 얼마나 오래 실행될지 알 수 없기 때문에, 동기화 영역에서 호출되면 다른 스레드는 그동안 보호된 자원을 사용하지 못하고 대기하게 된다.

<br>

## 3. 동시성 컬렉션 라이브러리

> ObservableSet.java

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

* ArrayList를 구현한 클래스로, 내부 변경 작업은 항상 깨끗한 복사본을 만들어 수행한다.
* 내부 배열은 수정되지 않으니 순회할 때 락이 필요 없어 빠르다.
* 수정할 일은 드물고 순회가 빈번한 경우 최적이다.

<br>

## 4. 정리

기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다. 동기화가 초래하는 진짜 비용은 락을 얻는 데 드는 시간이 아닌, 경쟁하느라 낭비하는 시간이다.

* 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.
* 가상머신의 코드 최적화를 제한한다는 점 또한 과도한 동기화의 비용이다.

가변 클래스를 작성하려거든 다음 두 선택지 중 하나를 따르자.

* 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 한다.
  * java.util
* 동기화를 내부에서 수행해 스레드 안전한 클래스로 만든다.
  * 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 고려한다.
  * java.util.concurrent
* StringBuffer와 Random을 동기화하지 않는 버전인 StringBuilder와 ThreadLocalRandom 등이 뒤늦게 등장했다.
* 선택이 어렵다면 동기화하지 않고 문서에 스레드 안전성을 명시한다.

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화해야 한다.

* 클라이언트가 여러 스레드로 복제돼 구동되는 상황이라면 다른 클라이언트에서의 호출을 막을 수 없어 외부 동기화가 불가능하다.
  * 정적 필드가 private이어도 서로 관련 없는 스레드들이 동시에 읽고 수정하는 등 전역 변수가 되버린다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
