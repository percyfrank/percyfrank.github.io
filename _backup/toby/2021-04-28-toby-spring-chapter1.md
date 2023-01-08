---
title: "[토비의 스프링] 1장. 오브젝트와 의존관계"
excerpt: "개발자는 미래의 변화에 유연하게 대처할 수 있는 방향으로 객체를 설계해야 한다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
toc: true
toc_sticky: true
last_modified_at: 2021-04-28
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter1)

## 1. 관심사의 분리

사용자의 비즈니스 프로세스 및 요구사항은 끊임없이 변한다. 따라서 개발자는 미래의 변화에 유연하게 대처할 수 있는 방향으로 객체를 설계해야 한다. 절차지향 프로그래밍에 비해 객체지향 프로그래밍은 추상화를 통해 변화에 효과적으로 대처할 수 있다. 가장 좋은 방법은 분리와 확장을 고려하여 설계하는 것이며, 이를 통해 변화로 인한 파급효과를 최소화한다.

변화는 대체로 집중된 한 가지 관심에서 일어나지만 그에 따른 작업은 한 곳에 집중되지 않은 경우가 많다. 따라서 관심이 같은 것들은 하나의 객체로 최대한 응집시키고, 관심이 다른 것은 객체들간 최대한 분리시켜 서로 영향을 주지 않도록 설계한다.

### 1.1. UserDao 예제

> UserDao.java

```java
public void addUser(User user) throws ClassNotFoundException, SQLException {
    Connection connection = getConnection();
    //DB 조작 로직 생략
}

public User getUser(String id) throws SQLException, ClassNotFoundException {
     Connection connection = getConnection();
     //DB 조작 로직 생략
}

private Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.cj.jdbc.Driver");
    return DriverManager.getConnection("jdbc:mysql://localhost:13306/tobi", "root", "root");
}
```

* ``getConnection()`` 메서드 분리를 통해 DB 연결 로직과 DB 조작 로직 관심사를 분리했다.
  * 한 가지 관심에 대한 변경이 발생하면 관심이 집중되는 코드만 수정한다.
* 그러나 DB 벤더 및 연결 방식을 수정하기 위해서는 Dao 클래스 내부 코드를 수정해야 한다.
  * Dao 클래스 내부 구현을 공개하지 않고 연결 방식을 외부에서 수정할 수는 없을까?

> UserDao.java

```java
public void addUser(User user) throws ClassNotFoundException, SQLException {
    Connection connection = getConnection();
    //DB 조작 로직 생략
}

public User getUser(String id) throws SQLException, ClassNotFoundException {
     Connection connection = getConnection();
     //DB 조작 로직 생략
}

public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
```

* UserDao를 추상 클래스로 선언하고 Connection을 생성하는 로직을 추상 메서드로 남겨둔다.
* UserDao를 상속하는 하위 클래스가 구체적인 DB 연결 방식을 구현하게끔 둔다.
  * 클래스 계층 구조를 통해 두 개의 관심이 분리된다.
* 템플릿 메서드 패턴이란?
  * 변하지 않는 기능은 슈퍼 클래스에 두고 자주 변경되어 확장할 기능은 추상 메서드 혹은 오버라이딩이 가능한 protected 메서드로 둔다.
  * 서브 클래스에서 필요에 맞게 해당 메서드들을 구현하도록 한다.
* 팩토리 메서드 패턴이란?
  * 하위 클래스에서 반환 타입인 인터페이스의 구체적인 오브젝트 생성 방식을 결정하게 한다.
* 그러나 [상속의 단점](https://xlffm3.github.io/java/item18-composition/)이 존재한다.

> UserDao.java

```java
public class UserDao {

    private ConnectionMaker connectionMaker; //구현체가 아닌 인터페이스

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    public void addUser(User user) throws ClassNotFoundException, SQLException {
        Connection connection = connectionMaker.makeConnection();
        //로직 생략
    }
}
```

* 변화의 성격이 다르다는 것은 변화의 이유와 주기 등이 다르다는 의미다.
  * 변화의 성격이 다른 관심들을 분리해서, 서로 영향을 주지 않은 채로 각각 필요한 시점에 독립적으로 개발되도록 한다.
* 인스턴스 변수로 구현체가 아닌 인터페이스를 사용하는 이유?
  * 특정 구현체를 사용하면 DB 연결을 가져오는 구체적인 방법에 종속되어, 자유롭게 확장(변경)하기 어렵다.
    * 기존의 구현체를 사용하던 코드 부분에서 수정이 수반되기 마련이다.
* 추상화란 공통적인 성격을 뽑아 분리하는 작업이다.
  * 인터페이스는 실제 구현체의 클래스의 정보를 캡슐화한다.
  * 구현체의 정보를 모르더라도 인터페이스를 사용하는 쪽은 공개 API만을 통해 통신하기 때문에, 구현체를 자유롭게 확장(변경)할 수 있다.
    * 구현체를 변경해도 인터페이스를 사용하는 기존 코드 부분에서는 수정이 수반되지 않는다.
  * **다형성**
* 생성자를 주입받는 이유?
  * 자신이 사용할 오브젝트를 Dao 내에서 new 연산을 통해 직접 관계를 맺는 것은 Dao의 책임, 관심사가 아니다.
  * 필드 변수의 구현체를 내부에서 직접 생성하면 다른 관심사로 인해 확장에 유연하지 못하다.
  * 외부 클라이언트가 런타임 오브젝트 관계를 맺어주는 책임을 가진다.
* [SOLID 패턴과 응집도 및 결합도에 대한 정리](https://xlffm3.github.io/design%20pattern/solid-pattern/)

### 1.2. 제어의 역전(IoC)

> DaoFactory.java

```java
public class DaoFactory {

    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    public AccountantDao accountantDao() {
        return new AccountantDao(connectionMaker());
    }

    public ConnectionMaker connectionMaker() {
        return new SimpleConnectionMaker();
    }
}
```

* 팩토리는 객체의 생성 방법을 결정하고 생성된 인스턴스를 반환한다.
  * 객체를 사용하는 책임과 만드는 책임을 분리한다.
* Dao나 ConnectionMaker가 로직을 담당하는 컴포넌트라면, Factory는 컴포넌트의 구조와 관계를 정의한 설계도다.
* IoC란 프로그램의 제어 흐름 구조가 뒤바뀌는 것이다.
  * 초기의 예제 코드는 오브젝트들이 필요한 오브젝트를 스스로 결정하고 필요한 시점에 생성해서 사용한다.
  * 리팩토링한 코드는 오브젝트가 사용할 오브젝트를 능동적으로 선택하지 않고 주입받는다.
    * 심지어 본인이 언제 어떻게 어디서 생성되고 사용되는지를 알지 못한다.
  * 엔트리 포인트를 제외한 오브젝트들은 제어 권한을 위임받은 특별한 오브젝트에 의해 결정되고 만들어진다.
    * DaoFactory라는 IoC 컨테이너가 UserDao 등 여러 컴포넌트의 생성을 제어한다.
  * 컴파일타임의 코드에는 의존성이 표현되어 있지 않으며, 런타임에 의존성이 주입된다.

#### 1.2.1. 프레임워크와 라이브러리

라이브러리를 사용하는 어플리케이션 코드는 어플리케이션의 흐름을 직접 제어한다. 반면 프레임워크는 어플리케이션 코드가 프레임워크에 의해 사용된다. 프레임워크가 흐름을 주도하는 상황에서 개발자가 작성한 코드를 사용하도록 하는 방식이다.

즉, 프레임워크는 IoC 개념이 도입되어 있다. 어플리케이션의 코드는 프레임워크 상에서 수동적으로 동작해야 한다. IoC 구현을 위해서는 컴포넌트의 생성과 관계 및 생명 주기 등을 관리하는 컨테이너 혹은 프레임워크가 필요하다.

<br>

## 2. Spring의 IoC

Bean이란 Spring이 제어권을 가지고 직접 만들며 관계를 부여하는 오브젝트다. Spring 프레임워크의 IoC 오브젝트는 BeanFactory다. ApplicationContext는 BeanFactory 인터페이스를 확장한 인터페이스로, 어플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당한다. ApplicationContext는 오브젝트들의 생성정보와 연관관계에 관한 외부 설정 정보를 참고하여 어플리케이션의 핵심 컴포넌트들을 구성한다.

> DaoFactory.java

```java
@Configuration
public class DaoFactory {

    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new SimpleConnectionMaker();
    }
}
```

* 오브젝트 설정 메타정보 클래스라고 인식할 수 있도록 @Configuration 및 @Bean 애너테이션을 부착한다.
  * XML과 같은 스프링 전용 설정정보다.

> UserDaoTest.java

```java
public class UserDaoTest {

    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao userDao = applicationContext.getBean("userDao", UserDao.class);
        User user = new User();
        //테스트 로직
    }
}
```

* DaoFactory라는 외부 설정 정보를 참고하여 ApplicationContext가 Bean들을 관리한다.
* ``getBean()`` 메서드가 굳이 Bean 반환 메서드 이름을 입력받는 이유?
  * 똑같은 UserDao Bean을 반환하더라도, 메서드별로 생성하는 방식과 구성을 다양하게 조성하기 위함이다.

### 2.1. ApplicationContext의 장점

* 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.
  * Spring을 사용하기 전의 코드는 특정 오브젝트를 사용하기 위해 어떤 팩토리 클래스를 사용해야 하는지 알아야한다.
  *   팩토리 클래스를 항상 생성해야 하는 번거로움이 존재한다.
  * ApplicationContext는 팩토리 클래스를 일단 등록만 해두면 일관된 방식으로 Bean을 가져와 사용할 수 있다.
    * ``public AnnotationConfigApplicationContext(Class<?>... componentClasses)``
* 애플리케이션 컨텍스트는 단순한 BeanFactory 기능을 넘어 종합 IoC 서비스를 제공해준다.
  * Bean의 등록, 생성, 조회, 반환 등 기본적인 관리 기능 이외에도 오브젝트가 만들어지는 방식, 시점, 전략, AOP, 인터셉터 등과 관련된 다양한 기능을 제공한다.
* 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.

<br>

## 3. 싱글톤 레지스트리

별다른 설정을 하지 않으면 Spring은 Bean 오브젝트들을 모두 싱글톤으로 만든다. Spring은 Java EE 기술을 사용하는 서버 환경에서 주로 적용되며, 클라이언트 요청을 처리하기 위해 표현-응용-데이터 등 다양한 계층의 객체들이 참여한다. 클라이언트 요청을 처리할 때마다 해당 객체들을 매번 새로 생성한다면, 동시다발적인 요청 부하가 심해질 때 서버가 감당하기 힘들다. 따라서 엔터프라이즈 분야에서는 **서비스 오브젝트**라는 개념을 일찍부터 사용해왔다.

* 서블릿은 대표적인 서비스 오브젝트로서, 스펙에서 강제하지는 않지만 대부분의 멀티 스레드 환경에서 싱글톤으로 작동한다.
  * 서블릿 클래스당 하나의 오브젝트만 만들고, 여러 스레드에서 공유해 동시에 사용한다.
* 일반적인 디자인 패턴에서의 [싱글톤](https://xlffm3.github.io/java/item3-singleton/)은 상속 불가, 테스트 어려움, 전역 상태 공유, 리플렉션 등을 통한 싱글톤 보장 실패 등의 한계가 존재한다.

Spring은 Bean들을 싱글톤으로 관리하지만, static 메서드와 private 생성자를 사용하는 비정상적인 클래스가 아닌 평범한 Java 클래스를 싱글톤으로 활용하게 해준다는 장점이 있다.

* 싱글톤으로 관리되는 Bean들은 정상적인 public 생성자를 가질 수 있다.
  * 싱글톤으로 관리되는 상황이 아니라면 간단하게 객체를 생성할 수 있다.
  * 테스트 환경에서는 자유롭게 Bean을 생성하고 목 객체로 대체할 수 있다.

### 3.1. 싱글톤 유의사항

```java
public class UserDao {

    private ConnectionMaker connectionMaker;
    private Connection c;
    private User user;

    public User get(String id) throws ClassNotFoundException, SQLException {
        this.c = connectionMaker.makeConnection();
        ...
        this.user = new User(); this.user.setId(rs.getString("id"));
        this.user.setName(rs.getString("name"));
        this.user.setPassword(rs.getString("password"));
        return this.user;
    }
}
```
* 싱글톤은 멀티 스레드 환경을 고려하여 상태 정보를 가지고 있지 않는 무상태 방식으로 설계되어야 한다.
  * ConnectionMaker와 같이 읽기 전용 값이라면 상태값으로 저장되어도 문제되지 않는다.
* 여러 스레드가 동시에 값을 새로 작성하는 Connection이나 User 상태를 가지고 있으면 안 된다.
  * 개별적으로 변하는 정보는 로컬 변수 혹은 메서드 파라미터 및 리턴값 등을 통해 다뤄야 한다.
  * 로컬 변수는 매번 새로운 값을 저장할 독립적인 공간이 만들어지며, 여러 스레드가 변수의 값을 덮어쓸 일은 없다.
* Scope란 Bean이 생성되고 존재하고 적용되는 범위를 의미하며, Spring은 싱글톤 이외에도 프로토타입 및 HTTP 관련 스코프를 제공한다.

<br>

## 4. DI

Spring IoC의 대표적인 동작 원리는 DI로 요약이 가능하다.

> UserDao.java

```java
public class UserDao {

    private ConnectionMaker connectionMaker; //구현체가 아닌 인터페이스

    //...
}
```

* UserDao는 구현체가 아닌 ConnectionMaker라는 인터페이스에 의존하고 있으며, 인터페이스가 변경되지 않는한 구현체가 변경되어도 UserDao 코드에는 변화가 없다.
  * 즉, 인터페이스는 의존관계를 느슨하게 만든다.
* 런타임 시에 오브젝트 사이에서 만들어지는 의존 관계를 런타임 의존 관계라고 한다.
  * 컴파일타임의 코드 및 클래스 모델에 의존 관계가 드러나지 않으며, UserDao는 런타임 시 인터페이스의 구현체에 대한 정보를 알지 못한다.
  * 런타임 시점의 의존 관계는 컨테이너 및 팩토리 같은 제3의 존재가 결정한다.
* Spring을 사용하기 전에는 DaoFactory 클래스가 DI 컨테이너의 역할을 수행했다.
* DI는 사용할 객체의 선택 및 생성 제어권을 외부에 넘기며, 자신은 주입받은 객체를 수동적으로 사용한다.
  * IoC 개념과 일맥상통한다.

### 4.1. 의존관계 검색

* 의존 관계를 맺는 방법이 외부로부터의 주입이 아니라 스스로 검색을 이용한다.
  * 자신이 필요로 하는 객체를 능동적으로 찾지만, 어떤 클래스의 객체를 사용할지 결정하지는 않는다.
  * 런타임 의존 관계를 맺을 객체 결정 및 생성은 외부 컨테이너에게 맡기지만, 이를 가져올 때는 주입이 아닌 컨테이너에게 요청한다.

> UserDao.java

```java
public UserDao() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
    this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}    
```

* 객체 스스로가 IoC 컨테이너에게 사용할 오브젝트를 요청한다.
* 좋은 패턴일까?
  * 사용자에 대한 DB 정보를 어떻게 가져올 것인가에 집중해야 할 UserDao에 여러 관심사가 산재하게 된다.
    * IoC 컨테이너나 팩토리를 만들어 API를 이용하는 코드가 섞인다.
  * 또한 공식 문서에서는 도메인 모델에 프레임워크의 코드가 침투하게 되면 결합도가 증가하기 때문에 직접 ``getBean()`` 호출을 지양하자고 한다.
  * 대게는 DI를 사용하는 편이 낫다.
* 하지만 사용이 필요할 때가 있다.
  * ``main()`` 메서드와 같은 엔트리 포인트에서는 DI가 불가능하며, 적어도 한 번은 사용할 오브젝트를 검색해야 한다.
  * 서버의 경우?
    * ``main()`` 메서드와 유사한 서블릿에서 Spring 컨테이너에 담긴 오브젝트를 사용하려면 한 번은 의존 관계 검색 방식을 사용한다.
* 의존 관계 검색 방식에서는 검색하는 객체는 Spring Bean일 필요가 없다.
  * 위 코드의 경우 UserDao는 Bean일 필요가 없다.
  * 반면 UserDao와 ConnectionMaker 사이에 DI가 적용되려면 UserDao도 Bean이어야 한다.

### 4.2. 유스케이스

* 개발할 때는 ConnectionMaker Bean의 구현체를 LocalDB 관련 클래스를 이용하다가, 운영 시에는 ProductionDB 관련 클래스로 수정한다.
  * 개발 시와 운영 시에 각각 다른 런타임 의존 관계를 형성해줌으로써, 코드 한 줄 수정으로 DB를 편하게 변경할 수 있다.

> CountingConnectionMaker.java

```java
public class CountingConnectionMaker implements ConnectionMaker {

    private final ConnectionMaker realConnectionMaker;
    private int counter = 0;

    public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
        this.realConnectionMaker = realConnectionMaker;
    }

    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        counter++;
        return realConnectionMaker.makeConnection();
    }

    public int getCounter() {
        return counter;
    }
}
```

> CountingDaoFactory.java

```java
@Configuration
public class CountingDaoFactory {

    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new CountingConnectionMaker(realConnectionMaker());
    }

    @Bean
    public ConnectionMaker realConnectionMaker() {
        return new SimpleConnectionMaker();
    }
}
```

> UserDaoTest.java

```java
public static void main(String[] args) throws SQLException, ClassNotFoundException {
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(CountingDaoFactory.class);
    UserDao userDao = applicationContext.getBean("userDao", UserDao.class);
    //...
    CountingConnectionMaker countingConnectionMaker = applicationContext.getBean("connectionMaker", CountingConnectionMaker.class);
    System.out.println(countingConnectionMaker.getCounter());
}
```

* 인터페이스를 활용한 DI를 통해 부가 기능을 간단하게 추가할 수 있다.
* 부가 기능을 활용한 ConnectionMaker 또한 UserDao가 사용하는 인터페이스를 구현하고 있기 때문에, 기존 DAO 설정 부분은 변경이 필요 없다.
  * CountingDaoFactory 및 CountingConnectionMaker를 더이상 사용하지 않는다면 해당 부분의 의존 관계 코드만 수정하면 된다.

> DaoFactory.java

```java
@Configuration
public class DaoFactory {

    @Bean
    public UserDao userDao() {
        UserDao userDao = new UserDao();
        userDao.setConnectionMaker(connectionMaker());
        return userDao;
    }
}
```

* 생성자가 아닌 수정자로 의존성 주입이 가능하다.

<br>

## 5. XML을 통한 설정

XML 파일의 Bean 설정 시, class 속성은 실제 Bean 오브젝트의 구현 클래스라는 점에 유의하자. 참조 변수 타입을 기입해서는 안 된다.

> ApplicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    <bean id="userDao" class="springbook.dao.user.UserDao">
        <property name="connectionMaker" ref="connectionMaker"/>
    </bean>
    <bean id="connectionMaker" class="springbook.dao.connection.SimpleConnectionMaker"/>
</beans>

```

* Java Bean 관례에 따라 수정자 메서드가 property다.
  * name 속성은 수정자 메서드에서 set을 제외한 나머지 부분이다.
* ref 속성은 수정자 메서드를 통해 주입해줄 의존 Bean 객체의 id다.
  * Bean의 id를 수정하는 경우, 이를 참조하고 있는 ref 속성들의 값도 함께 변경되어야 한다.
* XML 문서가 미리 정해진 구조에 따라 작성되었는지 검사하기 위해, 별도의 스키마 파일에 정의되어 있는 네임 스페이스를 사용한다.
  * beans 태그를 기본 네임 스페이스로 사용하는 스키마 선언은 위 예시처럼 작성한다.

> UserDaoTest.java

```java
public static void main(String[] args) throws SQLException, ClassNotFoundException {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserDao userDao = applicationContext.getBean("userDao", UserDao.class);
    //...
}
```

<br>

## 6. DataSource 추상화

Java는 DB 커넥션을 가져오는 객체의 기능을 추상화한 DataSource 인터페이스가 존재한다. 기본적으로 제공되는 여러 DataSource 구현체를 이용해보자.

> DaoFactory.java

```java
@Configuration
public class DaoFactory {

    @Bean
    public UserDao userDao() {
        UserDao userDao = new UserDao();
        userDao.setDataSource(dataSource());
        return userDao;
    }

    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSource simpleDriverDataSource = new SimpleDriverDataSource();
        simpleDriverDataSource.setDriverClass(com.mysql.cj.jdbc.Driver.class);
        simpleDriverDataSource.setUrl("jdbc:mysql://localhost:13306/tobi");
        simpleDriverDataSource.setUsername("root");
        simpleDriverDataSource.setPassword("root");
        return simpleDriverDataSource;
    }
}
```

* UserDao의 의존 객체를 ConnectionMaker에서 DataSource로 수정해주고, 설정 클래스 또한 부합되게 수정한다.

> ApplicationContext.xml

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:13306/tobi"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</bean>
```

* Bean을 정의할 때, 의존 객체가 아닌 단순 값도 주입할 수 있다.
  * DB 연결 정보가 변경되더라도 클래스 코드는 변하지 않는다.
* value 속성 값으로 지정된 String은 수정자 메서드의 파라미터 타입을 참조하여 자동으로 형변환이 된다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
