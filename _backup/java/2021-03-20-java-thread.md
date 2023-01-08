---
title: "Java Thread 기초 정리"
excerpt: "Thread의 개요와 핵심 개념들 및 사용 방법을 정리한다."
categories:
  - Java
tags:
  - Java
date: 2021-03-20
last_modified_at: 2021-03-20
---

## 1. Process vs Thread

프로세스란 **실행 중인 프로그램**이다. 프로그램을 실행하면 OS로부터 실행에 필요한 자원인 메모리를 할당받아 프로세스가 된다.

프로세스는 프로그램을 수행하는 데 필요한 데이터와 메모리 등의 자원 및 쓰레드로 구성된다. 프로세스의 자원을 이용해서 실제로 작업을 수행하는 것이 쓰레드이다.

모든 프로세스에는 최소한 하나 이상의 쓰레드가 존재하며, 둘 이상의 쓰레드를 가진 프로세스를 멀티쓰레드 프로세스라고 한다.

프로세스가 가질 수 있는 쓰레드의 개수는 제한되어 있지 않으나, 쓰레드가 작업을 수행하는데 개별적인 메모리 공간(호출 스택)을 필요로 하기 때문에 프로세스의 메모리 한계에 따라 생성할 수 있는 쓰레드의 수가 결정된다.

### 1.1. Multi-Tasking vs Multi-Threading

OS는 멀티 태스킹을 지원하기 때문에, 여러 개의 프로세스가 동시에 실행될 수 있다.

멀티 쓰레딩은 하나의 프로세스 내에서 여러 쓰레드가 동시에 작업을 수행하는 것이다. CPU의 코어는 한 번에 단 하나의 작업만 수행할 수 있으므로, 실제로 동시에 처리되는 작업의 개수는 코어의 개수와 일치한다. 그러나 쓰레드를 통해 코어가 아주 짧은 시간 동안 여러 작업을 번갈아 가며 수행함으로써, 여러 작업들이 모두 동시에 수행되는 것처럼 보이게 된다.

프로세스의 성능은 단순히 쓰레드의 개수에 비례하지 않는다.

### 1.2. Multi-Threading 장단점

* 장점
  * CPU의 사용률을 향상시킨다.
  * 자원을 보다 효율적으로 사용할 수 있다.
  * 사용자에 대한 응답성이 향상된다.
  * 작업이 분리되어 코드가 간결해진다.

여러 사용자에게 서비스를 해주는 서버 프로그램의 경우, 싱글 쓰레드라면 사용자의 요청 마다 새로운 프로세스를 생성해야 한다. 프로세스를 생성하는 것은 쓰레드를 생성하는 것에 비해 더 많은 시간 및 메모리 공간이 필요하기 때문에, 많은 수의 사용자 요청을 서비스하기 어렵다.

그러나 멀티 쓰레딩은 여러 쓰레드가 같은 프로세스 내에서 자원을 공유하기 때문에, 동기화 및 교착상태 등의 사이드 이펙트가 발생할 수 있다.

<br>

## 2. Thread 구현 및 실행

Thread 클래스를 상속받거나, Runnable 인터페이스를 구현한다. Runnable 인터페이스를 구현하는 방법이 재사용성이 높다. 두 방식은 Thread 인스턴스 생성 방법에 있어서 차이가 있다.

> Main.java

```java
public class Main {

    public static void main(String[] args) {
        MyThread myThread = new MyThread(); //Thread 클래스 상속
        myThread.start();

        Runnable runnable = new MyThread2();
        Thread myThread2 = new Thread(runnable);
        myThread2.start();
    }
}
```

Thread 클래스는 내부적으로 Runnable 인터페이스 참조변수를 가지고 있다. ``start()`` 메서드를 통해 쓰레드를 실행할 수 있다.

쓰레드를 통해 수행할 작업 내용은 다음과 같이 정의할 수 있다.
  * Thread 클래스를 상속받을 경우, ``run()``을 오버라이드하여 작업 내용을 정의한다.
  * Runnable 인터페이스를 구현하는 경우, ``run()`` 추상 메서드를 구현하여 작업 내용을 정의한다.

상속보다는 Runnable 인터페이스 구현을 통해 Thread에 ``run()``을 주입하는 방식이 보다 객체지향적인 방법이다.

생성자 및 setter 등을 통해 쓰레드의 이름을 지정할 수 있다.

한 번 실행된 쓰레드는 다시 실행될 수 없기 때문에, 작업을 한 번 더 수행해야 한다면 새로운 쓰레드 객체를 생성해야 한다. 또한 ``start()``가 호출되더라도 바로 실행되지 않고 실행 대기 상태에 머물다가, OS 스케쥴링에 의거하여 자신의 차례가 되었을 때 비로소 실행된다.

<br>

## 3. start() 및 run()

``start()`` 메서드는 새로운 쓰레드가 작업을 실행하는데 필요한 Call Stack을 생성한 다음, 생성된 호출 스택에 ``run()`` 메서드가 첫 번째로 올라가게 한다. 쓰레드가 종료되면 작업에 사용된 호출 스택은 소멸된다.

쓰레드는 작성된 OS 스케쥴에 따라 순서를 부여받고 작업을 수행하기 때문에, 쓰레드의 호출 스택에서 가장 위에 위치한 메서드 ``run()``은 실행 중이 아닌 대기 상태에 놓일 수 있다.

main 메서드 역시 main 쓰레드 위에서 작동되는 메서드이다. 실행 중인 사용자 쓰레드(User Thread)가 하나도 없을 때, 프로그램은 종료된다.

<br>

## 4. Single Thread vs Multi-Thread

쓰레드가 1개이든 2개이든, 작업 소요 시간은 거의 비슷하다. 오히려 멀티 쓰레드는 쓰레드 간의 작업 전환(context switching) 및 대기 시간으로 인해 시간이 더 소요된다.

싱글 코어인 경우 멀티 쓰레드라도 하나의 코어가 번갈아가면서 작업을 수행하기 때문에, 두 작업이 절대 겹치지 않는다. 그러나 멀티 코어의 경우 동시에 두 쓰레드가 수행될 수 있기 때문에, 자원을 놓고 경쟁하게 된다.

두 쓰레드가 서로 다른 자원을 사용하는 작업의 경우에는 싱글 쓰레드 프로세스보다 멀티 쓰레드 프로세스가 효율적이다.

Java가 OS 독립적이라고 하지만, 실제로 쓰레드는 OS에 종속적인 부분이다. JVM의 쓰레드 스케줄러에 의해서 어떤 쓰레드가 얼마동안 실행될 것인지 결정되는 것 처럼, Java 프로그램(프로세스) 또한 OS 프로세스 스케줄러에 영향을 받는다.

<br>

## 5. Thread 우선 순위

쓰레드는 우선 순위에 해당되는 속성을 가지고 있으며, 해당 값에 따라 쓰레드가 얻는 실행 시간이 달라진다. 쓰레드의 우선 순위는 쓰레드를 생성한 쓰레드로부터 자동 상속받는다. setter를 통해 인위적으로 우선 순위를 지정할 수 있으며, 숫자가 높을 수록 우선 순위가 높아진다.

그러나 이는 싱글 코어의 경우에만 보장된다. 멀티 코어의 환경에서는 OS마다 다른 방식으로 스케쥴링하기 때문에, 쓰레드 실행 결과가 OS마다 상이할 수 있다. 오히려 쓰레드의 우선 순위에 따른 차이가 없을 수 있다.

쓰레드에 우선 순위를 부여하는 대신 작업에 우선 순위를 두어 PriorityQueue에 저장해 놓고, 우선 순위가 높은 작업이 먼저 처리되도록 하는 것이 나을 수 있다.

<br>

## 6. ThreadGroup

서로 관련된 쓰레드를 그룹으로 다루기 위한 것이다. 쓰레드 그룹에 다른 쓰레드 그룹을 포함할 수 있다.

보안상의 이유로 도입된 개념이다. 자신이 속한 쓰레드 그룹이나 하위 쓰레드 그룹은 변경할 수 있지만, 다른 쓰레드 그룹의 쓰레드를 변경할 수는 없다.

쓰레드를 쓰레드 그룹에 포함시키려면 Thread 생성자를 이용한다.

```java
Thread(ThreadGroup threadGroup, String name)
//중략
```

모든 쓰레드는 반드시 쓰레드 그룹에 포함되어 있어야 하며, 쓰레드 그룹을 지정하지 않은 쓰레드는 기본적으로 자신을 생성한 쓰레드와 같은 그룹에 속하게 된다.

Java 어플리케이션이 실행되면 JVM은 main과 system이라는 쓰레드 그룹을 생성하고, JVM 운영에 필요한 쓰레드들을 생성해서 이 그룹에 포함시킨다. (예 : GC가 동작하는 Finalizer 쓰레드 등)

우리가 생성하는 쓰레드의 그룹을 지정하지 않으면 자동으로 main 쓰레드 그룹에 속하게 된다.

> Main.java

```java
public class Main {

    public static void main(String[] args) {
        ThreadGroup main = Thread.currentThread().getThreadGroup();
        ThreadGroup group1 = new ThreadGroup("group1");
        ThreadGroup group2 = new ThreadGroup("group2");
        ThreadGroup subGroup = new ThreadGroup(group1, "subgroup");

        group1.setMaxPriority(3);

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        new Thread(group1, runnable, "th1").start();
        new Thread(subGroup, runnable, "th2").start();
        new Thread(group2, runnable, "th3").start();

        System.out.println("Thread Name " + main.getName());
        System.out.println("Active Thread Group Counts" + main.activeGroupCount());
        System.out.println("Active Thread Counts" + main.activeCount());
        main.list();
    }
}
```

``list()`` 메서드를 통해 해당 쓰레드 그룹의 세부 정보를 출력한다.

<br>

## 7. Daemon Thread

데몬 쓰레드는 일반 쓰레드의 작업을 돕는 보조적인 역할을 수행하는 쓰레드이다. 일반 쓰레드가 모두 종료되면 데몬 쓰레드는 자동 종료된다.

데몬 쓰레드의 예로는 가비지 컬렉터, 워드 프로세서의 자동 저장, 화면 자동 갱신 등이 있다.

데몬 쓰레드는 무한 루프와 조건문을 이용해서 실행 후 대기하고 있다가, 특정 조건이 만족되면 작업을 수행하고 다시 대기되도록 작성한다.

데몬 쓰레드의 작성 및 생성 방법은 일반 쓰레드와 다르지 않으며, ``void setDaemon(boolean on)``을 호출하면 된다. 또한 데몬 쓰레드가 생성한 쓰레드는 자동적으로 데몬 쓰레드가 된다.

<br>

## 8. Thread 실행 제어

![image](https://user-images.githubusercontent.com/56240505/72275798-fbaf5e80-3671-11ea-9b9d-faae4e9bc5f9.png)<br><br> ![image](https://user-images.githubusercontent.com/56240505/72275824-0d910180-3672-11ea-9687-6d3141486a65.png)<br><br> ![image](https://user-images.githubusercontent.com/56240505/72275858-21d4fe80-3672-11ea-9b1a-fd44a822c9cf.png)<br><br>

쓰레드는 주어진 실행 시간이 다 되거나 ``yield()``를 만나면 다시 실행 대기 상태가 되고, 다음 차례의 쓰레드가 실행상태가 된다.

실행을 모두 마치거나 ``stop()``이 호출되면 쓰레드는 소멸된다.

``stop()``은 쓰레드를 종료시키고, ``suspend()``는 실행 상태의 쓰레드를 중지시키고, ``resume()``은 정지 상태의 쓰레드를 다시 실행 대기 상태로 만든다. 그러나 세 메서드들은 교착 상태 이슈로 deprecated 되었다.

### 8.1. sleep(long millis)

나노 세컨드까지 세밀하게 값을 지정할 수 있다. 일시정지 상태가 된 쓰레드는 지정된 시간이 다 되거나 ``interrupt()``가 호출되면 InterruptedException이 발생하며 잠에서 깨어나 실행대기 상태가 된다.

```java
void delay(long millis) {
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
    }
}
```

항상 try-catch를 사용하는 것 보다 별도의 메서드를 사용하기도 한다.

``sleep()``은 항상 현재 실행 중인 쓰레드에 작동하기 때문에, 참조변수 호출이 아닌 static 호출을 사용하자.

### 8.2. interrupt() 및 interrupted()

``interrupt()``는 쓰레드의 ``boolean interrupted``의 상태를 false에서 true로 변경한다. 쓰레드를 강제로 종료시키지는 못한다.

``interrupted()``는 쓰레드의 ``boolean interrupted``의 상태를 반환한 후, false로 변경한다.

쓰레드가 일시정지 상태(WAITING)일 때, 해당 쓰레드에 대해 ``interrupt()``를 호출하면, InterruptedException이 발생하고 쓰레드는 실행대기 상태(RUNNABLE)로 바뀐다.

``sleep()``가 종료되고 InterruptedException이 발생할 때, 쓰레드의 interrupted의 상태는 자동으로 false로 초기화된다. 따라서, ``interrupt()``를 호출하더라도 쓰레드가 ``sleep()``을 사용하고 있다면 ``isInterrupted()``가 true가 아닌 false가 나올 수 있다. catch block에서 ``interrupt()``를 다시 호출해주어야 원하는 결과가 나온다.

### 8.3. yield()

쓰레드 자신에게 주어진 실행 시간을 다음 차례의 쓰레드에게 양보한다. ``yield()`` 및 ``interrupt()``를 잘 활용하면, 프로그램의 응답성을 높이고 보다 효율적인 실행이 가능하다.

### 8.4. join()

쓰레드 자신이 하던 작업(실행)을 멈추고, 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 일시정지 상태(WAITING)이 된다.

시간을 지정하지 않으면, 해당 쓰레드가 작업을 모두 마칠 때까지 기다리게 된다. 작업 중에 다른 쓰레드의 작업이 먼저 수행되어야할 필요가 있을 때 사용한다.

``join()`` 또한 ``interrupt()``를 통해 대기 상태에서 벗어날 수 있으므로, try-catch가 필요하다. 다만 ``sleep()``과 다르게 현재 쓰레드가 아닌 특정 쓰레드에 동작하므로 static 메서드가 아니다. 쉽게 말해, 자신의 작업 중간에 다른 쓰레드의 작업을 참여(join)시키는 것이다.

<br>

## 9. Thread Synchronization

멀티 쓰레드 프로세스는 여러 쓰레드가 하나의 프로세스의 자원을 공유한다. 문제는 A 쓰레드가 작업하는 도중 작업 제어권이 B 쓰레드로 넘어가면서, A 쓰레드가 작업 중이던 데이터를 B 쓰레드가 변경하는 상황이다. 다시 A 쓰레드가 제어권을 획득하여 작업을 마치더라도, 의도하지 않은 결과를 얻을 수 있다.

이처럼, 쓰레드는 서로의 작업에 영향을 줄 수 있다. 이를 방지하기 위해 한 쓰레드가 특정 작업을 끝마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 **임계 영역**과 **Lock** 개념이 도입되었다.

Lock을 획득한 쓰레드만이 공유 데이터를 사용하는 임계 코드 영역에서 작업을 진행할 수 있고, Lock을 반납해야 다른 쓰레드가 해당 Lock을 획득해 임계 영역의 코드를 수행할 수 있다. 한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 동기화라고 한다.

### 9.1. synchronized

synchronized 키워드를 메서드에 붙이거나, 코드 일부를 block으로 감싼 뒤 그 앞에 ``synchronized(참조변수)``를 붙여주면 된다. Lock의 획득과 반납은 임계 영역의 진입 및 탈출 시 자동으로 이루어진다.

모든 객체는 Lock을 하나씩 가지고 있으며, 해당 객체의 Lock을 가지고 있는 쓰레드만 임계 영역의 코드를 수행할 수 있다. 다른 쓰레드들은 Lock을 얻을 때까지 기다리게 된다.

### 9.2. wait() 및 notify()

synchronized를 통해 공유 데이터를 보호하는 것은 좋으나 특정 쓰레드가 한 객체의 Lock을 너무 오래 가지고 있으면, 다른 쓰레드들은 해당 객체의 락을 기다리느라 원활한 작업이 진행되지 않는다.

임계 영역의 코드를 수행하다가 작업을 더 이상 진행할 상황이 아니면, 쓰레드가 Lock을 반납하도록 한다. 그러면 다른 쓰레드가 Lock을 얻어 해당 객체에 대한 작업을 수행할 수 있게 된다. 이후 작업을 진행할 상황이 되면 작업을 중단한 쓰레드가 Lock을 얻어 작업을 재진행한다. ``wait()``을 통해 Lock을 반납하고, ``notify()``를 통해 임계 영역에 재진입할 수 있다.

``wait()``이 호출되면 실행 중이던 쓰레드는 해당 객체의 대기실(waiting pool)에서 통지를 기다린다. ``wait()``은 ``notify()`` 및 ``notifyAll()``이 호출될 때까지 기다리지만, 매개변수가 있는 ``wait()``은 지정된 시간이 지난 후 자동적으로 ``notify()``가 호출된다. waiting pool은 객체마다 존재하기 때문에, ``notifyAll()``이 호출되더라도 모든 객체의 waiting pool에 있는 쓰레드가 깨워지는 것이 아니다. 호출된 객체의 waiting pool에 대기 중인 쓰레드만 해당된다.

해당 메서드들은 Object에 정의되어 있으며, 동기화 블록 내에서만 사용이 가능하다.

문제는 waiting pool에서 ``notify()``가 호출되었을 때, 어떤 쓰레드가 통지받을 지 알 수 없다. 따라서 원하는 쓰레드가 통지를 받지 못하고 오랫동안 대기 상태에 빠지는 기아(starvation) 현상이 발생할 수 있다. ``notifyAll()``을 사용하면 해당 객체의 waiting pool의 모든 쓰레드에게 통지하기 때문에, 원하는 쓰레드가 Lock을 얻어 작업을 할 수 있다. 하지만 불필요한 쓰레드들까지 통지를 받아 원하는 쓰레드와 Lock을 얻기 위해 경쟁하는 경쟁(race) 상태가 유발된다.

### 9.3. Lock과 Condition을 이용한 Synchronization

기존의 synchronized 블럭은 사용이 편리하지만 같은 메서드 내에서만 Lock을 걸 수 있다는 제약이 존재한다. 이럴 때 java.util.concurrent.locks 패키지가 제공하는 lock 클래스들을 이용하여 동기화를 할 수 있다.

* ReentrantLock
  * 재진입이 가능한 Lock으로 가장 일반적인 배타 Lock이다.
  * 무조건 Lock이 있어야만 임계 영역의 코드를 수행할 수 있다.
* ReentrantReadWriteLock
  * 읽기에는 공유적이고, 쓰기에는 배타적인 Lock이다.
  * 읽기 Lock이 걸려있으면, 다른 쓰레드가 읽기 Lock을 중복해서 걸고 읽기를 수행한다.
    * 읽기는 내용을 변경하지 않기 때문이다.
  * 그러나 읽기 Lock이 걸린 상태에서 쓰기 Lock을 거는 것은 허용되지 않으며, 반대 역시 허용되지 않는다.
* StampedLock
  * ReentrantReadWriteLock에 낙관적인 Lock의 가능을 추가했다.
  * 읽기 Lock이 걸려있으면 쓰기 Lock을 얻기 위해 읽기 Lock이 풀릴 때 까지 기다려야하는데 비해, 낙관적 읽기 Lock은 쓰기 Lock에 의해 바로 풀린다.
  * 낙관적 읽기에 실패하면, 읽기 Lock을 얻어서 다시 읽어 와야 한다.
  * 무조건 읽기 Lock을 걸지 않고, 쓰기와 읽기가 충돌할 때만 쓰기가 끝난 후에 읽기 Lock을 건다.

ReentrantLock은 임계 영역 전후로 ``lock()``과 ``unlock()``을 호출해주면 그만이다. 다만 예외 케이스들을 고려해서 확실한 Lock 반환을 위해, try - finally 혹은 synchronized 블록을 사용한다. 만약 다른 쓰레드에 의해 Lock이 걸려있을 때, Lock을 얻으려고 기다리지 않거나 지정된 시간만큼만 기다리도록 하는 ``tryLock()``을 사용할 수 있다.

``wait()`` 및 ``notify()``를 통해 쓰레드를 구현할 때, 쓰레드의 종류를 구분하지 않고 공유 객체의 waiting pool에 몰아넣어 기아 현상 혹은 경쟁 상태가 발생하게 된다. 따라서 특정 쓰레드를 위한 Condition을 만들고, 각각의 waiting pool에 따로 기다리도록 하면 문제를 해결할 수 있다.

```java
private ReentrantLock lock = new ReentrantLock();
private Condition forCook = lock.newCondition();
private Condition forCust = lock.newCondition();

private void test() {
    forCook.await(); //wait()
    forCust.signal(); //notify()
}
```

Lock을 통해 Condition을 생성하고, ``wait()`` 및 ``notify()``에 상응하는 메서드를 각 Condition 인스턴스를 통해 호출함으로써 대기와 통지의 대상을 명확하게 구분할 수 있다. Condition을 더욱 세분화하면 같은 종류의 쓰레드간의 기아 현상 및 경쟁 상태의 발생 가능성을 더 줄일 수 있다.

### 9.4. volatile

코어는 메모리에서 읽어온 값을 캐시에 저장하고, 캐시에서 값을 읽어서 작업한다. 다시 같은 값을 읽어올 때는 먼저 캐시에 있는지 확인하고 없을 때만 메모리에서 읽어온다.

도중에 메모리에 저장된 변수의 값이 변경되었는데도 캐시에 저장된 값이 갱신되지 않아, 메모리에 저장된 값과 다른 경우가 발생한다. 이러한 경우 원하지 않는 결과를 초래할 수 있다. 변수 앞에 ``volatile`` 키워드를 붙이면, 코어가 변수의 값을 읽어올 때 캐시가 아닌 메모리에서 읽어오기 때문에 캐시와 메모리간의 값 불일치가 해결된다.

synchronized 블럭을 사용해도 같은 효과를 얻을 수 있다. 쓰레드가 synchronized 블록에 진입하거나 나올 때, 캐시와 메모리간의 동기화가 발생한다.

JVM은 데이터를 4 byte 단위로 처리하기 때문에, int와 int보다 작은 타입들은 한 번에 읽거나 쓰기가 가능하다. 그러나 long 및 double 등은 8 byte이기 때문에, 변수의 값을 읽거나 쓰는 과정에서 다른 쓰레드가 끼어들 여지가 있다. 이 때, synchronized 블록보다 편한 방법은 해당 변수를 선언할 때 ``volatile`` 키워드를 붙이는 것이다.

* 상수는 변하지 않는 값이기 때문에 멀티 쓰레드에도 안전해서 ``volatile``을 붙일 수 없다.

``volatile``은 해당 변수에 대한 읽기 작업을 더 이상 나눌 수 없도록 원자화할 뿐, 동기화하는 것은 아니다.

<br>

## 10. fork & join Framework

fork & join Framework로 하나의 작업을 작은 단위로 나눠서 여러 쓰레드가 동시에 처리하는 것을 쉽게 만들어 준다.

* RecursiveAction : 반환값이 없는 작업을 구현할 때 사용한다.
* RecursiveTask : 반환값이 있는 작업을 구현할 때 사용한다.

두 클래스 중에서 하나를 상속받아 ``compute()`` 추상 메서드를 구현하면 된다. 이후 쓰레드풀과 수행할 작업을 생성하고 실행하면 된다.

```java
ForkJoinPool pool = new ForkJoinPool();
SumTask task = new SumTask(from, to);
//public class SumTask extends RecursiveTask<Long>
Long result = pool.invoke(task); //Thread의 start() 메서드처럼 사용
```

ForkJoinPool은 지정된 수의 쓰레드를 생성해서 미리 만들어 놓고 반복 재사용이 가능하다. 쓰레드 풀은 쓰레드가 수행해야 할 작업이 담긴 큐를 제공하며, 각 쓰레드는 자신의 작업 큐에 담긴 작업을 순서대로 처리한다. 기본적으로 코어의 개수와 동일한 개수의 쓰레드를 생성한다.

``compute()`` 메서드 내부에서 수행할 작업을 ``fork()`` 및 ``join()``을 통해 나눈다.

* ``fork()``
  * 작업을 쓰레드 풀의 작업 큐에 넣는 비동기 메서드.
* ``join()``
  * 작업의 수행이 끝날 때 까지 기다렸다가, 수행이 끝나면 그 결과를 반환하는 동기 메서드.

작업의 크기를 충분히 작게 나누어야 여러 쓰레드가 골고루 작업을 나누어 처리할 수 있다. 자신의 작업 큐가 비어있는 쓰레드는 다른 쓰레드의 작업 큐에서 작업을 가져와 수행하는 것을 쓰레드풀이 자동으로 관리한다.

<br>

---

## References

* Java의 정석 (남궁성 저) Chapter 13
