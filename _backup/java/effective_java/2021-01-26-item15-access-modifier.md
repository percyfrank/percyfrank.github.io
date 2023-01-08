---
title: "[Effective Java] Item 15. 클래스와 멤버의 접근 권한을 최소화하라"
excerpt: "캡슐화란 소프트웨어 설계의 근간이 되는 원리다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-01-26
last_modified_at: 2021-01-26
---

## 1. 캡슐화

컴포넌트는 모든 내부 구현을 숨김으로써 구현과 API를 분리한다. 오직 API를 통해 다른 컴포넌트와 소통할 뿐, 서로의 내부 동작 방식은 신경쓰지 않는다.

컴포넌트들을 독립시키는 캡슐화의 장점은 다음과 같다.

* 시스템 개발 속도를 높인다.
  * 여러 컴포넌트를 병렬로 개발할 수 있다.
* 시스템 관리 비용을 낮춘다.
  * 각 컴포넌트를 빨리 파악하여 디버깅이 쉽고, 컴포넌트 교체 부담이 적다.
* 성능 최적화에 기여한다.
  * 다른 컴포넌트에 영향을 주지 않고 특정 컴포넌트를 최적화할 수 있다.
* 소프트웨어 재사용성을 높인다.
  * 독자적으로 동작하는 컴포넌트는 낯선 환경에서도 유용하게 쓰일 가능성이 높다.
* 큰 시스템을 제작하는 난이도를 낮춘다.
  * 시스템 전체가 완성되지 않아도 개별 컴포넌트의 동작을 테스트할 수 있다.

<br>

## 2. 접근 제한자

모든 클래스와 멤버의 접근성을 가능한 좁혀야 한다. 멤버 클래스가 아닌 일반 클래스는 아래의 접근 제한자들 중 public과 package-private(default)만 사용 가능하다.

* private : 멤버를 선언한 톱 레벨 클래스에서만 접근할 수 있다.
* package-private(default) : 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있다.
  * 접근 제한자를 명시하지 않을 때 자동 적용되지만, 인터페이스의 멤버는 기본적으로 public이다.
* protected : package-private 접근 범위를 포함하며, 해당 멤버를 선언한 클래스를 상속한 다른 패키지의 하위 클래스에서도 접근이 가능하다.
* public : 모든 패키지에서 접근 가능하다.

### 2.1. API

톱 레벨 클래스나 인터페이스를 public으로 공개하면 API가 되기 때문에 하위 호환을 위해 계속 관리해야 한다. package-private으로 선언하면 API가 아닌 내부 구현이 되어 언제든 수정할 수 있기 때문에, 외부 패키지에 공개할 필요가 없다면 package-private을 사용한다.

### 2.2. Inner Class

한 클래스에서만 사용하는 package-private 톱 레벨 클래스 혹은 인터페이스는 사용하는 클래스 내부에 private static Inner Class로 중첩할 수 있다. Outer Class를 제외하면 해당 클래스에 접근할 수 없게 된다.

### 2.3. 멤버

클래스의 공개 API를 설계하고 그 외의 모든 멤버는 private으로 선언한다. 패키지의 다른 클래스가 접근해야 하는 멤버에 한해 package-private으로 풀어준다. **protected**는 공개 API로서 영원히 지원되어야 하고 문서에 공개해야 할 수도 있기 때문에 적을 수록 좋다.

<br>

## 3. 제약

상위 클래스의 메서드를 오버라이드할 때 접근 수준을 상위 클래스보다 더 좁게 설정할 수 없다. 이는 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체될 수 있다는 리스코프 치환 원칙을 지키기 위함이다.

<br>

## 4. public 관련 고려사항

public 클래스의 인스턴스 필드는 가급적 public이 아니어야 한다.

* final이 아닌 필드 및 final이더라도 가변 객체를 참조하는 필드 등이 public이면 외부에서 수정이 가능하기 때문이다.
  * 이는 필드와 관련된 불변식이 깨지고, 스레드에도 안전하지 못하다.

다만 해당 클래스가 표현하는 추상 개념을 완성하는데 꼭 필요한 구성 요소로써의 상수는 public static final로 공개해도 좋다. 이런 필드는 반드시 불변 객체를 참조해야 한다. 다른 객체를 참조하지 못하더라도, 참조하는 객체 자체가 변경될 수 있기 때문이다.

> Test.java

```java
public static final Thing[] PRIVATE_VALUES = { ... };
```

배열은 final이더라도 수정이 가능하다. 따라서 해당 필드에 직접 접근할 수 있는 API를 공개하면 안 된다.

> Test.java

```java
private static final Thing[] PRIVATE_VALUES = { ... };

public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

// 혹은

public static final Thing[] values() {
     return PRIVATE_VALUES.clone();
}
```

외부에서 변경을 가할 수 있는 필드는 불변 컬렉션으로 제공하거나 방어적 복사본을 제공하는 API를 사용하도록 한다.

<br>

## 5. 모듈

Java 9 부터는 모듈의 개념이 도입되었다. 모듈이란 패키지의 묶음으로서 자신에 속한 패키지 중 공개할 것을 지정한다.

* protected 혹은 public 멤버라도 해당 패키지를 공개하지 않았다면 모듈 외부에서 접근할 수 없다.
  * protected와 public의 효력이 모듈 내부로 제한된다.
* JAR 파일을 자신의 모듈 경로가 아닌 클래스패스에 위치시키면 모듈의 효과가 사라진다.
  * 패키지 공개 여부에 상관없이, 모듈 외부에서 모듈 내부 패키지의 public 및 protected 멤버에 접근 가능해진다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
