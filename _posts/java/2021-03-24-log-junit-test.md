---
title: "JUnit을 이용한 Log 메시지 테스트"
excerpt: "ListAppender를 통해 출력되는 Log 메시지를 검증해보자."
categories:
  - Java
tags:
  - Java
date: 2021-03-24
last_modified_at: 2021-03-24
---

## 1. 의존성

> build.gradle

```groovy
dependencies {
    compile group: 'org.slf4j', name: 'slf4j-api', version: '1.7.25'
    compile group: 'ch.qos.logback', name: 'logback-classic', version: '1.3.0-alpha5'
}
```

<br>

## 2. 테스트 코드

> TestClassTest.java

```java
@DisplayName("NumberTest 클래스에서 Test 애너테이션이 붙은 메서드들을 검증한다.")
@Test
void runTests() throws ClassNotFoundException {
    //given
    ListAppender<ILoggingEvent> listAppender = new ListAppender<>();
    Logger logger = (Logger) LoggerFactory.getLogger(TestCase.class);
    logger.addAppender(listAppender);
    listAppender.start();

    //when
    TestClass testClass = TestClass.from(Class.forName("test.domain.NumberTest"));
    testClass.runTests();
    List<ILoggingEvent> testLogs = listAppender.list;
    String message = testLogs.get(0).getFormattedMessage();
    Level level = testLogs.get(0).getLevel();

    //then
    assertThat(message).isEqualTo("[Test Failed] :: NumberTest - compareFailed");
    assertThat(level).isEqualTo(Level.INFO);
    assertThat(testLogs).hasSize(7);
}
```

LoggerFactory에서 검증하려는 로그가 발생하는 대상 클래스의 Logger 인스턴스를 가져온다. 이 때, 참조 변수 Logger는 ``org.slf4j.Logger``가 아닌 ``ch.qos.logback.classic.Logger``다.

* Logger에 ListAppender를 등록하고, ListAppender를 실행한다.
* 검증할 테스트를 수행한 다음, ListAppender에서 로깅 이벤트(ILoggingEvent)를 기록해둔 List를 반환받는다.
  * ILoggingEvent에 정의된 API를 통해 로그 메시지 및 레벨 등을 확인할 수 있다.

<br>

---

## References

* [How to do a JUnit assert on a message in a logger](https://stackoverflow.com/questions/1827677/how-to-do-a-junit-assert-on-a-message-in-a-logger)
