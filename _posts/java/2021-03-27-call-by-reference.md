---
title: "Call By Reference vs Call By Value"
excerpt: "Java는 Call By Value다."
categories:
  - Java
tags:
  - Java
date: 2021-03-27
last_modified_at: 2021-03-27
---

## 1. Call By Reference

* 참조에 의한 호출을 의미한다.
* 전달받은 값을 직접 참조하며, 전달받은 값을 변경할 경우 원본도 같이 변경된다.

<br>

## 2. Call By Value

* 값을 호출하는 것을 의미한다.
* 전달받은 값을 복사하여 처리하고, 전달받은 값을 변경하여도 원본은 변경되지 않는다.

<br>

## 3. Java는 Call By Value

> Test.java

```java
public static void change(Person a1, Person a2) {
    a1.age = 32;
    a2 = a1;
}

public static void main(String[] args) {
    Person a1 = new Person(10);
    Person a2 = new Person(20);
    change(a1, a2);
    System.out.println(a1.getAge(), a2.getAge()); //32, 20
}
```

Java에서 객체를 전달받고 해당 객체를 수정하면 원본도 같이 수정되니 Call By Reference라고 생각하기 쉽다. 결론부터 말하자면 Java는 모두 Call By Value다.

* Java는 매개변수를 넘기는 과정에서 직접적인 참조를 넘긴 게 아닌, 주소 값을 복사해서 넘긴다.
* 복사된 주소 값으로 참조가 가능하니 주소 값이 가리키는 객체의 내용 변경될 수는 있다.
* 그러나 값을 복사해서 넘기기 때문에 Call By Value다.

> Test.cpp

```c++
void change(Person* a1, Person* a2) {
	a1->value = 111;
	*a2 = *a1;
};


int main() {
	Person* a1 = &Person(1);
	Person* a2 = &Person(2);
	change(a1, a2);
	cout << "a1 : " << a1->value << ", a2 : " << a2->value; //111, 111
	return 0;
}
```

Call By Reference의 대표적인 예시인 C++ 코드다.

* 참조 변수의 주소값을 복사해서 주지 않고, 직접적인 참조를 건넨다.
* ``change()`` 함수 내부에서 a2의 참조에 a1을 대입했기 때문에, 외부에서 a2도 a1을 가리키게 된다.

<br>

---

## References

* [[Java] Java는 Call by reference가 없다](https://deveric.tistory.com/92)
