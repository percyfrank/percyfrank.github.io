---
title: "[토비의 스프링] 2장. 테스트"
excerpt: "테스트란 개발자가 마음 편하게 잠자리에 들 수 있게 해주는 것이다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
toc: true
toc_sticky: true
last_modified_at: 2021-05-16
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter2)

## 1. 테스트

어플리케이션은 계속 변하고 복잡해져 간다. 이에 잘 대응하기 위해서는 확장과 변화를 고려한 객체지향 설계가 중요하다. 아울러 개발자가 작성 및 수정한 코드가 의도한 대로 정확히 동작하는지 확신할 수 있는 **테스트** 작업도 수반되어야 한다.

### 1.1. 단위 테스트의 중요성

웹 어플리케이션의 경우 여러 Layer의 기능들을 제작한 다음, 어플리케이션을 서버에 배치하고 웹 화면을 띄워 입력값을 직접 테스트할 수 있다. 그러나 에러가 발생했을 때, 해당 에러가 어떤 Layer의 어느 부분에서 기인한 것인지 신속하게 파악 및 대응하기 어렵다.

* 따라서 테스트할 대상(관심)을 작은 단위로 분리하고 접근한다.
* 작은 단위의 코드에 대한 테스트를 단위 테스트라고 칭한다.
  * 단위의 크기 및 범위는 명문화된 것은 아니지만, 단위를 넘어서는 다른 코드들을 신경 쓰지 않아도 테스트가 동작할 수 있는 정도다.
  * 통제할 수 없는 외부의 리소스에 의존하는 테스트는 단위 테스트라 보기 어렵다.
* 단위 테스트 없이 통합 테스트만 진행하면 에러 상황에서 원인을 찾기 어렵다.
  * 신규 기능 개발 혹은 유지보수 등으로 어플리케이션 코드에 수정이 발생할 때, 변경에 영향을 받는 부분을 파악하기 어렵다.
  * 단위 테스트가 잘 작성된 경우라면?
    * 모든 테스트를 실행해서 성공하면, 새로 추가한 기능이 정상적으로 동작하고 기존의 기능에도 영향을 주지 않았다는 확신을 얻을 수 있다.
    * 테스트에 실패했다면, 실패 원인 지점을 신속하게 특정하여 대응할 수 있다.
* 단위 테스트는 지속적인 개선과 점진적인 개발을 가능하게 해준다.
* 단위 테스트는 코드가 바뀌지 않는다면 매번 실행할 때마다 일관된 결과를 보장해야 한다.
  * DB에 남아있는 데이터 등 외부 환경 및 테스트 순서에 영향받아서는 안 된다.

<br>

## 2. TDD

TDD(Test Driven Development)란 만들고자 하는 기능의 내용을 담은 테스트 코드를 먼저 작성하고, 기능을 구현하는 사이클의 개발 방법론이다. TDD를 통해 자연스럽게 단위 테스트를 작성할 수 있으며, 개발자가 구현한 기능에 대한 피드백을 빠르게 받고 확신을 가질 수 있다. 테스트 코드는 일종의 기능 정의서와 일맥상통한다.

<br>

## 3. JUnit

JUnit은 대표적인 Java 테스트 프레임워크다. 프레임워크는 개발자가 만든 클래스에 대한 제어 권한을 넘겨받아서 주도적으로 어플리케이션의 흐름을 제어한다. 개발자가 만든 테스트 클래스의 오브젝트를 생성하고 실행하는 일은 프레임워크에 의해 진행된다. 따라서 테스트 수행을 위해 main 메서드나 관련 객체를 직접 생성할 필요가 없다.

각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 보장하기 위해, JUnit은 @Test를 실행할 때마다 테스트 클래스 오브젝트를 새로 만든다. 따라서 가변하는 인스턴스 변수를 부담없이 사용할 수 있다.

### 3.1. Spring 테스트

테스트에서 사용할 ApplicationContext 픽스처를 @Before 메서드에서 new로 직접 생성하는 경우, 테스트 메서드 개수만큼 생성된다. 등록된 Bean이 많거나 별도의 초기화 작업을 거치거는 경우, ApplicationContext 생성에 많은 시간이 소요된다. 독자적으로 리소스 혹은 스레드를 점유하는 Bean의 경우, 테스트를 마칠 때마다 이를 반환하지 않으면 다음 테스트에 영향을 끼칠 수 있다.

ApplicationContext는 싱글톤 레지스터리로서, 한 번 초기화되고 나면 상태가 변경되는 일이 거의 없다. 따라서 ApplicationContext처럼 생성에 많은 시간 및 자원이 소비되는 경우, 테스트 전체가 공유하는 오브젝트로 만들 수 있다. 이를 통해 테스트 성능이 대폭 향상된다.

> UserDaoTest.java

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
class UserDaoTest {

    @Autowired
    private ApplicationContext applicationContext;
    @Autowired
    private UserDao userDao;
    //...
}
```

* @RunWith은 JUnit 프레임워크 테스트 실행 방법을 확장하는 애너테이션으로, JUnit이 테스트 진행 중 사용할 ApplicationContext를 만들고 관리하는 작업을 진행해준다.
* JUnit 확장 기능은 테스트 실행 전에 딱 한 번만 ApplicationContext를 생성해두고, 테스트 오브젝트가 만들어질 때마다 필드에 주입한다.
  * 여러 테스트 클래스 파일들이 동일한 설정 파일을 @ContextConfiguration으로 참조하고 있으면, 테스트 클래스 사이에서도 ApplicationContext를 공유한다.
* @Autowired가 붙은 변수에 대해, 테스트 컨텍스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 Bean을 찾아 주입해준다.
  * ApplicationContext 또한 Bean으로 등록되기 때문에 DI가 가능하다.
  * 변수에 할당 가능한 타입을 가진 Bean을 자동으로 찾기 때문에, 인터페이스 타입의 변수를 선언해도 된다.
    * 주입될 수 있는 같은 타입의 Bean이 두 개 이상인 경우, 변수명과 일치하는 Bean 주입을 시도하며, 선택할 수 없는 경우 예외가 발생한다.

<br>

## 4. 테스트와 DI

> UserDao.test

```java
@Autowired
DataSource dataSource;
```

* 테스트는 어플리케이션 클래스를 검증하기 위한 목적이기 때문에, 특정 타입과 밀접한 종속 관계를 가져도 된다.
* 하지만 꼭 필요하지 않다면 테스트에서도 인터페이스를 사용해 느슨한 연결을 지향한다.
  * 테스트에서 특정 DataSource 구현체가 아닌 인터페이스를 사용하는 경우를 생각해보자.
  * DataSource Bean의 구현 클래스가 변경되더라도, 테스트 코드를 수정할 필요가 없다.

### 4.1. 유의사항

> UserDaoTest.java

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
class UserDaoTest {

    @Autowired
    private ApplicationContext applicationContext;
    @Autowired
    private UserDao userDao;

    @BeforeEach
    void setUp() {
        DataSource dataSource = new SingleConnectionDataSource( "jdbc:mysql://localhost/testdb", "spring", "book", true);
        userDao.setDataSource(dataSource);
    }
    //...
}
```

* 현재 ``applicationContext.xml``에 등록된 DataSource Bean은 운영용 DB 커넥션을 반환한다.
  * 이를 테스트에서 사용해서는 안 된다.
* 테스트에서 dao 객체에게 테스트용 DataSource를 주입하는 방식을 사용할 수 있다.
  * 그러나 ``applicationContext.xml`` 파일 정보를 바탕으로 제작된 ApplicationContext의 Bean 상태를 강제로 변경시키는 행위이다.
  * ApplicationContext는 테스트 중에 딱 한 개만 만들어져 모든 테스트에서 공유되기 때문에, 상태 변경이 바람직하지 않다.
    * UserDao의 의존 관계가 가변된 ApplicationContext는 다른 테스트에서도 사용될 것이며, 테스트 순서에 따라 일관되지 않은 결과가 도출될 수 있다.

### 4.2. @DirtiesContext

@DirtiesContext는 Spring의 테스트 컨텍스트 프레임워크에게 ApplicationContext의 상태를 변경한다는 것을 알린다. 이에 따라, 테스트 컨텍스트는 해당 애너테이션이 붙은 테스트 클래스에서는 ApplicationContext의 공유를 허용하지 않는다. 테스트 중에 변경된 ApplicationContext가 뒤의 테스트에 영향을 주지 않게끔, 테스트가 수행되고 나면 새로운 ApplicationContext를 생성한다.

> UserDaoTest.java

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "/test-applicationContext.xml")
class UserDaoTest {

    @Autowired
    private ApplicationContext applicationContext;
    //...
}
```

그러나 이 방법 또한 ApplicationContext을 새롭게 생성한다는 단점이 존재한다. 테스트 코드에서 Bean 오브젝트에게 수동으로 DI하기 보다는, 테스트를 위한 별도의 설정 파일 ``test-applicationContext.xml`` 등을 두고 사용하는 것이 낫다. 테스트 전용 ApplicationContext 한 개를 모든 테스트에서 공유할 수 있다.

### 4.3. 컨테이너 없는 DI 테스트

> UserDaoTest.java

```java

public class UserDaoTest {

    private UserDao dao;

    @BeforeEach
    public void setUp() {
        //...
        dao = new UserDao();
        DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true); dao.setDataSource(dataSource);
    }
}
```

ApplicationContext와 같은 Spring 컨테이너에 의존하지 않고, 테스트 코드에서 직접 오브젝트를 만들고 DI해서 사용해도 된다. Spring은 어플리케이션 로직을 담은 코드에 영향을 주지 않고도 사용이 가능한 **비침투적 기술**이다. 기술에 종속적이지 않은 순수한 코드를 사용할 수 있기에, 컨테이너 없이 DI가 가능하다.

DI는 객체지향 프로그래밍 스타일일 뿐, 컨테이너가 필요한 것이 아니다. DI 컨테이너나 프레임워크는 DI를 편하게 적용하도록 도움을 줄 뿐이다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
