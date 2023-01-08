---
title: "[Effective Java] Item 87. 커스텀 직렬화 형태를 고려해보라"
excerpt: "자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용한다."
categories:
  - Java
tags:
  - Java
  - Effective Java
date: 2021-02-12
last_modified_at: 2021-02-12
---

## 1. 기본 직렬화 형태가 적합한 경우

Serializable을 구현하고 기본 직렬화 형태를 사용한다면 다음 릴리즈 때 버리려 한 구현에 발이 묶이게 된다. 기본 직렬화 형태를 버릴 수 없게 되기 때문이다. 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민하고 합당한 경우에만 사용한다.

* 객체의 기본 직렬화 형태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 효율적으로 인코딩한다.
  * 객체가 포함한 데이터들과 그 객체에서 시작해 접근할 수 있는 모든 객체 및 연결된 위상까지 기술한다.
* 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.

> Name.java

```java
public class Name implements Serializable {
    /**
    * 이름. null이 아니어야 함.
    * @serial
    */
    private final String lastName;
    private final String middleName;
    private final String firstName;  
}
```

* 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.
  * 성명은 논리적으로 3개의 문자열로 구성되며, 인스턴스 필드들이 이 논리적 구성요소를 잘 반영한다.
* 기본 직렬화 형태가 적합하더라도 불변식 보장과 보안을 위해 ``readObject()`` 메서드를 제공해야 할 때가 많다.
* private 필드임에도 직렬화 형태에 포함되는 공개 API기 때문에 Javadoc으로 문서화한다.

<br>

## 2. 기본 직렬화 형태가 적합하지 않은 경우

> StringList.java

```java
public final class StringList implements Serializable {

    private int size   = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry  next;
        Entry  previous;
    }
}
```

물리적으로 문자열들을 이중 연결 리스트로 연결하며, 논리적으로는 일련의 문자열을 표현한다. 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 Entry를 기록한다.

### 2.1. 문제점

* 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
  * private 클래스인 StringList.Entry가 공개 API가 되어, 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다.
  * 연결 리스트를 사용하지 않더라도 관련 코드를 삭제할 수 없다.
* 너무 많은 공간을 차지한다.
  * 엔트리와 연결 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다.
  * 직렬화 형태가 너무 커져서 디스크 저장 및 네트워크 전송 속도가 느리다.
* 시간이 많이 걸릴 수 있다.
  * 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없어 그래프를 직접 순회하게 된다.
* 스택 오버플로우를 일으킬 수 있다.
  * 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 특정 OS나 JVM에서는 문제가 될 수 있다.

<br>

## 3. 커스텀 직렬화

> StringList.java

```java
public final class StringList implements Serializable {

    private transient int size   = 0;
    private transient Entry head = null;

    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }
}
```

StringList의 물리적인 상세 표현을 배제한 채 논리적인 구성만 담는다. 리스트가 포함한 문자열의 개수와, 그 뒤로 문자열들을 나연하는 수준이면 된다.

* transient 한정자는 해당 인스턴스 필드가 직렬화에 포함되지 않는다는 표시다.
* 인스턴스 필드가 모두 transient라면 ``defaultReadObject()`` 및 ``defaultWriteObject()``를 호출할 필요는 없으나, 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가될 때의 호환을 대비해 호출하도록 한다.
  * 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화하면 새로 추가된 필드들은 무시된다.
* private 메서드 또한 직렬화로 인해 공개 API가 되기 때문에 문서화한다.

<br>

## 4. 주의사항

해시테이블에 기본 직렬화를 사용하면 심각한 버그로 이어질 수 있다.

* 해시테이블은 키-값 엔트리를 어떤 버킷에 담을지는 키에서 구한 해시코드가 결정한다.
  * 계산 방식은 구현에 따라 달라질 수 있으며, 계산할 때마다 달라지기도 한다.
* 따라서, 해시테이블을 직렬화한 후 역직렬화하면 불변식이 심각하게 훼손된 객체가 생겨날 수 있다.

기본 직렬화 수용 여부에 상관없이 ``defaultWriteObject()`` 메서드는 transient로 선언하지 않은 모든 인스턴스 필드를 직렬화한다.

* transient 선언이 가능한 인스턴스 필드에는 모두 해당 한정자를 붙여야 한다.
  * 캐시된 해시 값처럼 다른 필드에서 유도되는 필드들도 해당된다.
  * long 필드 등 JVM을 실행할 때마다 값이 달라지는 네이티브 자료구조를 가지는 필드들도 해당된다.
* 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.

기본 직렬화를 사용하면 transient 필드들은 역직렬화될 때 기본값으로 초기화된다.

* 기본값을 사용해서는 안 된다면 ``readObject()`` 메서드에서 ``defaultReadObject()``를 호출한 다음, 필드를 원하는 값으로 복원한다.
* 혹은 지연 초기화 방법을 사용한다.

기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.

* 객체의 모든 메서드가 synchronized로 스레드 안전하다면, 기본 직렬화 메서드 ``writeObject()``에 synchronized 선언을 해야한다.
  * 메서드 내부에서 동기화하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다.
  * 그렇지 않으면 자원 순서 교착 상태에 빠질 수 있다.

어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하라.

* 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 성능이 약간 저하될 수 있다.
* 자동 생성되는 UID 값에 의존하면 추후 변경사항이 발생할 때 UID 값이 함께 변경되어 런타임 에러가 발생하는 등 호환성 이슈를 유발한다.
  * 직렬 버전 UID가 없는 기존 클래스를 구버전으로 직렬화된 인스턴스와 호환성을 유지한 채 수정하고 싶다면, 구버전에서 사용한 자동 생성된 값을 그대로 사용해야 한다.
* 꼭 고유한 값일 필요는 없다.

기존 버전 클래스와 호환성을 끊고 싶다면 단순히 직렬 버전 UID 값을 변경하면 된다.

* 기존 버전의 직렬화된 클래스를 역직렬화할 때 InvalidClassException이 발생한다.
* 그 외의 경우는 수정해서는 안 된다.

<br>

---

## Reference

* Effective Java 3/E(Joshua Bloch 저)
