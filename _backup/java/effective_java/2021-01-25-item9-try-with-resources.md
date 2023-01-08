---
title: "[Effective Java] Item 9. try-finally보다 try-with-resources를 사용하라"
excerpt: "사용 자원을 자동으로 해제해주는 기능을 알아보자."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-25
last_modified_at: 2021-01-25
---

## 1. try-finally

사용한 자원을 제대로 닫아주기 위해 전통적으로 try-finally를 사용해왔다.

* finalizer는 안전망의 역할로 제한적으로 사용된다.

> Copy.java

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            if (out != null) out.close();
        }
    } finally {
        if (in != null) in.close();
    }
}
```

try-finally를 사용하여 자원을 닫아줄 때의 문제는 다음과 같다.

* 사용 자원이 여러 개인 경우 코드가 중첩되어 지저분해진다.
  * 예외 처리를 위해 finally 블럭에서 특정 자원이 null이 아닐 때 ``close()`` 하는 조건문 등.
* try 블록에서 예외가 발생했을 때 finally 블록의 ``close()``에서도 두 번째 예외가 발생하는 경우.
  * 두 번째 예외로 인해 첫 번째 예외 정보가 남지 않아 디버깅이 어려워진다.

<br>

## 2. try-with-resources

> TopLineWithDefault.java

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

AutoCloseable 인터페이스를 구현한 자원은 try-with-resources를 통해 간단하게 관리된다. try 블록의 작업이 완료되거나 예외가 발생했을 때 자동으로 ``close()``가 호출되어 자원이 해제된다.

만약 ``readLine()``과 ``close()`` 모두 예외가 발생하면 ``readLine()`` 예외가 기록된다. 그러나 ``close()`` 예외 정보는 버려지지 않고 ``readLine()`` 예외에 담아진(suppressed)된 상태이기 때문에 언제든 확인할 수 있다.

Java 8 까지는 try-with-resources를 사용하기 위해 try 블록의 소괄호에 자원을 할당해야 했다.

> TopLineWithDefault.java

```java
static String firstLineOfFile(String path, String defaultVal) {
    BufferedReader br1 = new BufferedReadernew(new FileReader(path));
    BufferedReader br2 = new BufferedReadernew(new FileReader(path));
    try (br1; br2) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

Java 9 부터는 try 블록 밖에서 선언된 객체를 참조할 수 있다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
