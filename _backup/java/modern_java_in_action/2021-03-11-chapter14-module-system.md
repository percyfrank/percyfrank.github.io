---
title: "[Modern Java in Action] 14장. Java 모듈 시스템"
excerpt: "Java 9에 들어서 클래스 파일과 패키지의 가시성을 제어하는 모듈화 기능이 도입되었다."
categories:
  - Java
tags:
  - Java
  - Modern Java in Action
date: 2021-03-11
last_modified_at: 2021-03-11
---

## 1. 압력 : 소프트웨어 유추

### 1.1. 관심사 분리

관심사 분리(SoC)는 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다. Java는 패키지를 통해 관련있는 클래스들을 그룹핑하지만 엄밀한 의미의 모듈화는 지원하지 않는다. 관심사 분리의 장점은 다음과 같다.

* 각각의 모듈을 독립적으로 개발이 가능하여 협업 생산성을 높인다.
* 각 모듈의 재활용성을 높인다.
* 시스템 전체의 유지보수가 용이해진다.

### 1.2. 정보 은닉

정보 은닉이란 구현에 대한 구체적인 내용을 숨길 것을 장려하는 원칙이다. 구현 내용을 숨김으로써 한 부분의 변화가 프로그램의 다른 부분에 변화에 영향을 줄 가능성을 낮춘다.

* Java는 private 접근 제어자를 통해 클래스의 구현 내용을 캡슐화할 수 있다.
  * 그러나 내부 구현을 목적으로 제작되어 공개를 원치 않는 패키지가 public 필드 혹은 메서드를 가지고 있다면, 구현 내용이 외부에 노출되는 단점이 있었다.
  * 원래 의도와 다르게 외부에서 이러한 패키지의 코드를 사용하는 경우 의존성이 생기며, 추후 코드를 변경하거나 유지보수할 때 유연하게 대처하기 어렵다.
  * ``sun.misc.Unsafe``와 같은 API는 JDK 내부 구현 목적으로 개발되었으나, 모듈화 지원 미비로 JDK 외부의 여러 라이브러리가 사용하고 있다.
    * 호환성을 유지하면서 해당 API를 변경하기 어려워진다.
* Java 9에 들어서 클래스 파일과 패키지의 가시성을 제어하는 모듈화 기능이 도입되었다.
  * 원래 의도와 부합한 범위에서만 공개되도록 패키지를 캡슐화한다.

<br>

## 2. Java 모듈 시스템

### 2.1. 설계 목적

Java는 컴파일된 클래스들을 하나의 JAR 파일로 만들며 클래스 패스에서 JAR에 접근한다. JVM은 클래스가 필요할 때마다 클래스 패스로부터 동적으로 클래스를 로드한다. 그러나 클래스 패스는 다음과 같은 단점이 있다.

* 클래스 패스는 같은 클래스에 대한 버전 정보를 인식하지 못한다.
  * 서로 다른 버전의 라이브러리 클래스가 클래스 패스에 여러 개 존재한다면 프로그램 동작 결과는 알 수 없다.
* 클래스 패스는 명시적인 의존성을 지원하지 않는다.
  * 여러 JAR에 속한 클래스 파일들은 클래스 패스에서 한 공간에 모이게 된다.
  * 클래스 패스는 어떤 JAR가 다른 JAR에 속한 클래스를 의존한다는 등을 명시할 수 없다.
  * Java 9 이전까지는 클래스 패스에 의존성이 누락되거나 충돌이 있는지 등을 확인하기 어려웠다.
    * 런타임 시점에 ClassNotFound 예외가 발생하기 전까지 모를 수도 있다.
    * 의존성 관리를 위해 Maven 및 Gradle 등의 빌드 툴을 주로 사용한다.
* Java 9 부터는 모듈 시스템의 도입으로 이러한 문제점을 컴파일 시점에 감지할 수 있다.

## 2.2. Java 모듈 시스템

``module-info.java`` 파일에 모듈 설명자를 서술함으로써 Java에서 모듈화를 설정할 수 있다. 모듈 설명자는 크게 두 가지 키워드로 구분된다.

* requires : 해당 모듈이 동작할 때 필요한 외부 모듈을 명시한다.
* exports : 다른 모듈이 사용할 수 있도록 해당 모듈 내 내부 패키지의 가시성을 명시한다.
* ``module-info.java``는 해당 모듈의 루트 위치에 있어야 한다.

<br>

## 3. 실사용 예제

> ProjectDirectory

```
|─ expenses.application
  |─ module-info.java
  |─ com
    |─ example
     |─ expenses
      |─ application
        |─ ExpensesApplication.java

|─ expenses.readers
  |─ module-info.java
    |─ com
      |─ example
        |─ expenses
          |─ readers
            |─ Reader.java
        |─ file
          |─ FileReader.java
        |─ http
          |─ HttpReader.java
```

* 위와 같은 디렉토리 구조의 프로젝트에서 application 모듈만 동작시켜보자.

> Script

```bash
javac module-info.java com/example/expenses/application/ExpensesApplication.java -d target

jar cvfe expenses-application.jar
    com.example.expenses.application.ExpensesApplication -C target

java --module-path expenses-application.jar \
     --module expenses/com.example.expenses.application.ExpensesApplication
```

* ``--module-path`` 옵션은 어떠한 모듈이 로드 가능한지 명시한다.
* ``--module`` 옵션은 동작시킬 메인 모듈과 클래스를 명시한다.

> module-info.java

```java
module expenses.readers {
    requires java.base;

    exports com.example.expenses.readers;
    exports com.example.expenses.readers.file;
    exports com.example.expenses.readers.http;
}
```

* requires는 모듈 네임을, exports는 패키지 네임을 명시한다
* ``java.base``는 Java의 기본 유틸리티(컬렉션, 입출력 등)를 포함하고 있는 모듈이며, 명시적으로 선언하지 않아도 기본으로 포함된다.

### 3.1. Maven 활용

> ProjectDirectory

```
|- pom.xml
|─ expenses.application
  |- pom.xml
  |─ module-info.java
  |─ com
    |─ example
     |─ expenses
      |─ application
        |─ ExpensesApplication.java

|─ expenses.readers
  |- pom.xml
  |─ module-info.java
    |─ com
      |─ example
        |─ expenses
          |─ readers
            |─ Reader.java
        |─ file
          |─ FileReader.java
        |─ http
          |─ HttpReader.java
```

* xml 파일은 교재를 참고하여 실습해보자.
* ``mvn clean package``를 실행하면 다음 두 개의 JAR가 생성된다.
  * ``./expenses.application/target/expenses.application-1.0.jar``
  * ``./expenses.readers/target/expenses.readers-1.0.jar``

> Scirpt

```bash
java --module-path \
     ./expenses.application/target/expenses.application-1.0.jar:\
     ./expenses.readers/target/expenses.readers-1.0.jar \
     --module \
     expenses.application/com.example.expenses.application.ExpensesApplication
```

### 3.2. 자동화 모듈

Maven에서 의존성 주입을 받은 외부 라이브러리를 ``module-info.java``의 requires 구에 추가한다. ``module-info.java``가 없지만 모듈 패스에 존재하는 JAR는 자동화 모듈이 된다. 자동화 모듈은 내부적으로 모든 패키지를 export한다. 자동화 모듈의 이름은 JAR 이름을 통해 자동으로 명명된다.

> Script

```bash
java --module-path \
     ./expenses.application/target/expenses.application-1.0.jar:\
     ./expenses.readers/target/expenses.readers-1.0.jar \
     ./expenses.readers/target/dependency/httpclient-4.5.3.jar \
     --module \
     expenses.application/com.example.expenses.application.ExpensesApplication
```

* 주입받은 httpClient 외부 라이브러리는 자동으로 모듈화가 된다.

### 3.3. 기타

``module-info.java``에서 사용할 수 있는 키워드는 다음과 같다. 필요할 때 각각의 키워드의 특징을 학습하고 사용해보자.

* requires
* exports
* requires transitive
* exports to
* open
* opens
* opens to
* uses
* provides

<br>

---

## Reference

* Modern Java in Action(Raoul-Gabriel Urma 저)
