---
title: "[Effective Java] Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라"
excerpt: "문서화 주석은 여러분 API를 문서화하는 가장 훌륭하고 효과적인 방법이다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-08
last_modified_at: 2021-02-08
---

## 1. Javadoc

API 문서는 코드가 변경되면 매번 함께 수정되어야 하는데, Javadoc 유틸리티는 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

* 직렬화할 수 있다면 직렬화 형태에 관해서도 적어야 한다.
* 기본 생성자에는 주석을 달 방법이 없으니, 공개 클래스는 기본 생성자를 사용하면 안 된다.
* 유지보수까지 고려한다면 공개되지 않은 counterpart들에도 문서화 주석을 달아야 할 것이다.

### 1.1. 메서드용 문서화 주석

메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

* 어떻게 동작하는지가 아닌 무엇을 하는지를 기술한다.
* 클라이언트가 해당 메서드를 호출하기 위한 전제조건을 모두 나열한다.
* 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건도 모두 나열한다.
  * @throws 태그로 비검사 예외를 선언하며, 비검사 예외 하나가 전제조건 하나와 연결된다.
  * @param 태그로 그 조건에 영향받는 매개변수에 기술할 수도 있다.
* 스레드 등 사후조건으로 명확히 나타나지 않는 부작용도 문서화해야 한다.

> BigInteger.java

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index) {
    return null;
}
```

* Javadoc 유틸리티는 문서화 주석을 HTML로 변환한다.
* {@code} 태그는 감싼 내용을 코드용 폰트로 렌더링하며, 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.
  * {@literal} 태그로 감싸면 코드용 폰트로 렌더링하지 않으나, 그 외는 {@code} 태그와 동일한 효과를 가진다.
* 인스턴스 메서드의 문서화 주석에 쓰인 this는 호출된 메서드가 자리하는 객체를 가리킨다.

### 1.2. 자기사용 패턴

> List.java

```java
/**
 * Returns true if this collection is empty.
 *
 * @implSpec This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() {
    return false;
}
```

* 클래스를 상속용으로 설계할 때는 자기사용 패턴에 대해서도 문서를 남긴다.
  * @implSpec 태그는 해당 매서드와 하위 클래스 사이의 계약을 설명한다.
  * 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 어떻게 동작하는지 서술한다.

### 1.3. 요약 설명

각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주된다. 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.

> Suspect.java

```java
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
 public enum Suspect {
    MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE
}
 ```

* 한 클래스 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안 된다.
* 요약 설명은 첫 문장에서 처음 발견되는 마침표까지만 해당되기 때문에 마침표 사용에 주의해야 한다.
  * 의도치 않은 마침표를 포함한 텍스트는 {@literal}로 감싼다.
* Java 10 부터는 {@summary} 요약 설명 전용 태그가 추가되었다.

메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 (주어가 없는) 3인칭 동사구여야 한다. 한편 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다.

<br>

## 2. 주석 작성 규약

> Fragment.java

```java
/**
 * This method complies with the {@index IEEE 754} standard.
 */
 ```

* Java 9 부터는 Javadoc이 생성한 HTML 문서에 검색(색인) 기능이 추가되어 쉽게 검색할 수 있다.
* {@index} 태그를 통해 API에서 중요한 용어를 추가로 색인화할 수 있다.

> ExceptionTest.java

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```

* 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.
* 애너테이션 타입의 요약 설명도 포함되어야 한다.
* 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.
* 열거 타입을 문서화할 때는 모든 상수들에 주석을 달아야 한다.

### 2.1. 기타

* API를 문서화할 때는 스레드 안전 수준과 직렬화 가능성을 API 설명에 포함해야 한다.
* 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성한다.
* Javadoc은 메서드 주석을 상속시킬 수 있다.
  * {@inheritDoc} 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.
  * 문서화 주석이 없는 API 요소들은 Javadoc이 가장 가까운 클래스 혹은 인터페이스의 문서화 주석을 찾아준다.
* 여러 클래스가 상호작용하는 API라면 문서화 주석 외에도 전체 아키텍처를 설명하는 별도의 설명이 필요하다.
  * 설명 문서가 있다면 클래스나 패키지의 문서화 주석에 링크를 추가한다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
