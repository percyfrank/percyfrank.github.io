---
title: "[번역] Compact Strings in Java 9"
excerpt: "JDK 9의 향상된 문자열 옵션에 대해 알아보자."
categories:
  - Java
tags:
  - Java
date: 2022-01-03
last_modified_at: 2022-01-03
---

## 1. 개요

Java String 클래스 내부는 char 타입 배열로 구현되어있다. Java는 내부적으로 UTF-16 인코딩을 사용하기 때문에, char는 2바이트로 구성된다. ASCII 문자는 1바이트로 표현될 수 있기 때문에, 영문자 1개로 구성된 String의 선행 8 비트는 전부 0이 된다.

많은 문자들이 컴퓨터 상에서 표현하기 위해 16비트를 요구하지만, 통계적으로 대부분은 8비트로 충분하다. 특히 LATIN-1 표기 문자들이 그러하다. 메모리 소모 및 성능을 향상할 여지가 존재한다는 뜻이다.

String은 일반적으로 JVM Heap 영역의 많은 부분을 차지한다. 또한 JVM에 문자열이 저장되는 방식으로 인해, 일반적으로 **String 인스턴스는 실제 필요한 것보다 두 배나 되는 공간을 차지한다.**

이번 글에서는 JDK 6에 도입된 Compressed String 및 JDK 9에 새롭게 도입된 Compact String에 대해 알아볼 것이다. 두 옵션 모두 JVM 상 문자열 메모리 소모를 최적화하는 것에 목적을 둔다.

<br>

## 2. Compressed String - Java 6

JDK 6는 하기와 같이 성능 향상을 위한 새로운 VM 옵션을 발표했다.

> Shell

```bash
-XX:+UseCompressedStrings
```

해당 옵션이 활성화되면 String은 char 배열이 아닌 byte 배열로 저장된다. 이를 통해 많은 메모리를 절약할 수 있으나, 의도치 않은 성능 저하로 인해 JDK 7에서 해당 옵션은 삭제되었다.

<br>

## 3. Compact String - Java 9

Java 9에서 압축된 String의 개념이 재도입되었다.

골자는 다음과 같다. **String 생성시, 문자열의 모든 문자가 LATIN-1 표기와 같이 1 바이트로 표현될 수 있다면 byte 배열이 내부적으로 사용된다는 것이다.**

만약 문자열 중 한 문자라도 표현에 필요한 비트가 8비트를 초과하는 경우, 모든 문자들은 UTF-16 표기를 위해 한 글자당 2 바이트 공간을 할당 받아 저장된다.

간단하게 말하자면, 가능한 경우 문자 표기를 위해 char가 아닌 byte를 사용한다는 것이다.

여기서 드는 의문은 다음과 같다.

* 모든 String 연산이 어떻게 동작하는가?
* JVM이 Latin-1 표기와 UTF-16 표기를 어떻게 구분하는가?

해당 이슈들을 해결하기 위해, String 내부 구현에 작은 변화가 생겼다. ``coder``라는 불변 필드가 바로 관련 정보를 담고 있다.

### 3.1. Java 9의 String 구현

예전까지 String은 char 배열로 저장되었다.

> String.java

```java
private final char[] value;
```

지금부터는 byte 배열로 저장될 것이다. 변수 coder가 String 내부에 존재하며, 변수 coder는 LATIN1 혹은 UTF16 값을 가진다.

> String.java

```java
static final byte LATIN1 = 0;
static final byte UTF16 = 1;

private final byte[] value;
private final byte coder;
```

대부분의 String 연산은 coder를 확인하고 세부 구현에게 위임된다.

> String.java

```java
public int indexOf(int ch, int fromIndex) {
    return isLatin1()
      ? StringLatin1.indexOf(value, ch, fromIndex)
      : StringUTF16.indexOf(value, ch, fromIndex);
}  

private boolean isLatin1() {
    return COMPACT_STRINGS && coder == LATIN1;
}
```

JVM이 필요한 모든 정보가 준비되고 사용가능하면, Compact String VM 옵션은 기본적으로 활성화된다. 해당 옵션을 끄려면 하기의 옵션을 사용한다.

> Shell

```bash
+XX:-CompactStrings
```

### 3.2. coder 동작 방식

Java 9 String 클래스에서 문자열 길이 계산은 다음과 같이 구현되었다.

> String.java

```java
public int length() {
    return value.length >> coder;
}
```

만약 String이 LATIN-1 문자만을 포함한다면, coder 값은 0이며 String 문자열 길이는 byte 배열의 길이와 동일하다.

만약 String에 UTF-16 표기 문자가 존재하면, coder 값은 1이며 문자열의 길이는 실제 byte 배열 크기의 절반이다.

Compact String 관련 변경 사항은 String 클래스 내부 구현에 반영된 사항임을 명심하라.

<br>

## 4. Compact String vs Compressed String

JDK 6의 Compressed String이 마주한 가장 큰 문제는 'String 생성자는 오직 char 배열 인자만을 받아들인다'는 점이다. 게다가 대부분의 String 연산은 byte 배열이 아닌 char 배열 표기에 의존 중이었다. 이러한 까닭에, 많은 unpacking 작업이 수반되었고 성능에 영향을 끼쳤다.

Compact String 또한 coder라는 별도의 필드를 유지하기 때문에 오버헤드가 증가한다. coder의 비용 및 UTF-16 표기 상황에서 byte를 char로 unpacking하는 비용을 완화하기 위해, 일부 메서드가 내장되었으며 JIT 컴파일러에 의해 생성되는 ASM 코드 또한 향상되었다.

이러한 변화는 일부 직관적이지 않은 결과를 낳기도 했다. LATIN-1 표기에서 ``indexOf(String)`` 메서드는 내장 함수를 호출하는 반면, ``indexOf(char)``는 그렇지 않다. UTF-16의 경우, 위 두 메서드는 내장 함수를 호출한다. 이러한 이슈는 LATIN-1 문자열에만 영향을 끼치며, 향후 JDK 릴리즈에서 수정될 것이다.

따라서 Compact String이 성능 측면에서 우수하다. Compact String을 사용할 때 메모리를 얼마나 절약하는지 확인하기 위해, 다양한 Java application의 heap 덤프가 분석되었다. 결과는 특정 어플리케이션에 과하게 의존적이었지만, 전반적으로 성능이 향상되었다.

### 4.1. 성능 차이

Compact String 사용 여부에 따른 성능 차이를 비교해보자.

> Test.java

```java
long startTime = System.currentTimeMillis();

List strings = IntStream.rangeClosed(1, 10_000_000)
  .mapToObj(Integer::toString)
  .collect(toList());

long totalTime = System.currentTimeMillis() - startTime;
System.out.println(
  "Generated " + strings.size() + " strings in " + totalTime + " ms.");

startTime = System.currentTimeMillis();

String appended = (String) strings.stream()
  .limit(100_000)
  .reduce("", (l, r) -> l.toString() + r.toString());

totalTime = System.currentTimeMillis() - startTime;
System.out.println("Created string of length " + appended.length()
  + " in " + totalTime + " ms.");
```

천 만개의 문자열을 생성하고 문자열을 조합한다. Compact String이 활성화되었을 때 다음과 같은 출력이 나온다.

> Shell

```
Generated 10000000 strings in 854 ms.
Created string of length 488895 in 5130 ms.
```

CompactString 옵션을 비활성화하고 동일하게 어플리케이션을 실행해본다.

> Shell

```
Generated 10000000 strings in 936 ms.
Created string of length 488895 in 9727 ms.
```

특정 상황에서 CompactString이 성능 개선에 영향을 줄 수 있다.

<br>

## 5. 기타

요약하자면 다음과 같다.

* Compact Strings는 byte[]을 채택함으로써 문자열에 따라 Latin-1(1byte) 혹은 UTF-16(2byte)로 인코딩된 문자를 저장한다.
* **이를 통해 메모리 공간 효율이 높아지고 GC 발생이 적어지는 성능 개선 효과를 누릴 수 있다.**

<br>

---

## References

* [JEP 254: Compact Strings](https://openjdk.java.net/jeps/254)
* [Compact Strings in Java 9](https://www.baeldung.com/java-9-compact-string)
* [5월 우아한 Tech 세미나 후기](https://techblog.woowahan.com/2627/)
