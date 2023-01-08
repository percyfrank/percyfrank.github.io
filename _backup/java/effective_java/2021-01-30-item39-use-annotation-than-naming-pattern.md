---
title: "[Effective Java] Item 39. 명명 패턴보다 애너테이션을 사용하라"
excerpt: "애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-30
last_modified_at: 2021-01-30
---

## 1. 명명 패턴의 단점

과거 JUnit은 테스트 메서드의 이름을 test로 시작하도록 작성해야 했다. 그렇지 않으면 테스트 도구는 테스트 메서드를 무시하고 지나쳤다.

* 명명 패턴은 오타가 나면 안 된다.
* 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.
* 프로그램의 요소를 매개변수로 전달할 마땅한 방법이 없다.
  * 예외가 발생하길 원하는 테스트는 메서드 이름에 예외 이름을 명시해야 했다.
  * 그러나 컴파일러는 메서드 이름에 덧붙인 문자열이 예외인지 알 도리가 없다.
  * 테스트를 실행하기 전까지는 해당 이름의 클래스가 존재하는지 혹은 예외인지 모른다.

그러나 이후 JUnit이 애너테이션 기반으로 변경됨에 따라 이러한 문제들이 해결되었다.

<br>

## 2. 마커 애너테이션

> Test.java

```java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

아무 매개변수 없이 단순히 대상에 마킹하기 때문에 마커 애너테이션이라고 한다. 애너테이션을 달았다고 특정 메서드나 클래스에 직접적으로 영향을 주지 못한다. 단지, **애너테이션에 관심있는 프로그램에게 추가 정보 및 특별한 처리를 할 기회를 제공한다.**

* @Retention과 @Target은 애너테이션 선언에 사용되는 메타 애너테이션이다.
  * @Retention은 해당 애너테이션이 언제까지 유지되는지 알려준다.
  * @Target은 해당 애너테이션이 어느 선언에서 사용되어야 하는지 알려준다.

> RunTests.java

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
    }
}
```

* 특정 클래스에 @Test 애너테이션이 붙은 메서드를 파악한다.
* 테스트 메서드가 예외를 던지면 리플렉션 메커니즘인 InvocationTargetException으로 감싸서 다시 던진다.
* 이 경우, 테스트 메서드가 static이 아니면 역시나 예외가 발생한다.

<br>

## 3. 매개변수를 받는 애너테이션

> ExceptionTest.java

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

* Throwable을 확장한 클래스를 허용하기 때문에 사실상 모든 예외를 수용한다.

> Sample.java

```java
public class Sample {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
```

* 애너테이션 옆에 예외 클래스를 명시해준다.

> RunTests.java

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf(
                    "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                    m, excType.getName(), exc);
        }
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

* 특정 예외의 클래스 파일이 컴파일 시점에 존재했으나 런타임 시점에 존재하지 않는 경우?
  * TypeNotPresentException이 발생한다.
* 타입 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인한다.

### 3.1. 배열 매개변수를 받는 애너테이션

> ExceptionTest.java

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

> Sample.java

```java
@ExceptionTest({ IndexOutOfBoundsException.class,
                   NullPointerException.class })
public static void doublyBad() { ... }
```

* 위와 같이 코드를 수정하면 애너테이션이 배열 매개변수를 받게 된다.

> RunTests.java

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
     tests++;
     try {
         m.invoke(null);
         System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
     } catch (Throwable wrappedExc) {
         Throwable exc = wrappedExc.getCause();
         int oldPassed = passed;
         Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
         for (Class<? extends Throwable> excType : excTypes) {
             if (excType.isInstance(exc)) {
                 passed++;
                 break;
             }
         }
         if (passed == oldPassed)
             System.out.printf("테스트 %s 실패: %s %n", m, exc);
     }
}
```

* 복수개의 발생 가능 예외들과 실제 발생한 예외를 비교하여 카운팅한다.

<br>

## 4. @Repeatable

애너테이션에 @Repeatable 애너테이션을 달면, 해당 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

* @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 정의해야 한다.
  * @Repeatable에 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
* 컨테이너 애너테이션은 내부 애너테이션의 타입의 배열을 반환하는 ``value()`` 메서드를 정의해야 한다.
* 적절한 @Retention과 @Target이 정의되어야 한다.

> ExceptionTest.java

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

> ExceptionTestContainer.java

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

> Sample.java

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { ... }
```

* 반복적으로 애너테이션을 달 수 있도록 지원해준다.
* 단, 반복 가능 애너테이션을 여러 개 달면 한 개만 달았을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.
  * ``getAnnotationsByType()`` 메서드는 이 둘을 구분하지 않고 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져온다.
* ``isAnnotationPresent()`` 메서드는 둘을 명확하게 구분한다.
  * 반복 가능 애너테이션이 달렸는지 검사할 때, 컨테이너 애너테이션을 단(반복 가능 애너테이션이 여러개 달린) 메서드는 무시한다.
  * 컨테이너 애너테이션을 검사할 때는, 반복 가능 애너테이션이 1개 달린 메서드는 무시한다.
  * 따라서 이 두 개를 모두 검사해야 한다.

> RunTests.java

```java
if (m.isAnnotationPresent(ExceptionTest.class)
        || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

애너테이션이 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다. 다만 애너테이션을 선언하고 처리하는 부분의 코드가 복잡해지게 된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
