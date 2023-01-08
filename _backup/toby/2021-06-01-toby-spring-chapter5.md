---
title: "[토비의 스프링] 5장. 서비스 추상화"
excerpt: "서비스 추상화 및 DI는 테스트를 편리하게 작성하는데 기여한다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
toc: true
toc_sticky: true
last_modified_at: 2021-06-01
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter5)

## 1. 사용자 레벨 관리 기능 추가

* 매직 넘버의 상수 추출 의의
  * 의미가 있는 매직 넘버는 여러 곳에서 자주 사용될 가능성이 높다.
  * 한 가지 변경 이유가 발생할 때, 여러 곳을 고치게 된다면 중복이다.
  * 상수로 추출하면 매직 넘버가 변경되더라도 이를 사용하는 모든 곳이 아니라 상수 선언 부분만 수정하면 된다.
* 객체지향코드의 의의
  * 다른 객체의 상태 정보를 가져와 비즈니스 로직을 처리하는 대신, 데이터를 가진 객체에게 작업을 해달라고 요청한다.
  * 객체들은 자신의 책임에 충실하니 전반적인 코드 이해가 쉬우며, 변경이 발생해도 수정 위치를 쉽게 특정할 수 있다.
  * 여러 로직이 한 곳에 산재해있으면 코드가 길어져 이해가 어려우며, 예외 상황을 특정하기 어렵다.
    * 즉, 변화에 취약하다.

<br>

## 2. 트랜잭션 서비스 추상화

트랜잭션이란 더 이상 나눌 수 없는 단위의 작업을 말한다. 작업을 쪼개서 작은 단위로 만들 수 없다는 것은 핵심 속성인 원자성을 의미한다. 트랜잭션이 성공한다면 커밋을, 중간에 예외가 발생해 작업을 완료할 수 없다면 아예 작업이 시작되지 않은 것처럼 초기 상태로 롤백해야 한다.

DB 자체는 완벽한 트랜잭션을 제공한다. 그러나 서비스 메서드에서 여러 개의 쓰기 작업 쿼리를 날리는 경우도 하나의 트랜잭션으로 동작해야 한다. 트랜잭션은 Connection 객체를 통해 시작과 종료를 지정할 수 있다.

트랜잭션은 Connection 객체를 통해 시작과 종료 등 경계를 지정할 수 있다. Jdbc는 기본적으로 하나의 작업이 마무리되면 자동으로 커밋이 적용된다. 따라서 여러 DB 작업이 하나의 트랜잭션으로 묶이지 않는다. 따라서 ``connection.setAutoCommit(false)``로 트랜잭션 시작을 선언하며, ``commit()``이나 ``rollback()``을 호출하여 종료를 선언한다. 즉, 트랜잭션의 경계는 하나의 Connection이 만들어지고 닫히는 범위에만 존재한다.

### 2.1. JdbcTemplate의 문제점

JdbcTemplate 메서드 호출시 Connection을 생성하며, 일련의 작업을 처리한 다음 Connection을 반환한다.  트랜잭션은 Connection의 범위보다 좁기 때문에, 템플릿을 사용하는 UserDao의 각 메서드들은 하나씩 독립적인 트랜잭션으로 수행된다.

즉, 유저들의 레벨 변경 중 작업 예외가 발생했을 때 기존에 업데이트 된 유저들은 이미 커밋되어 롤백되지 않는다. 트랜잭션이 아니기 때문이다. JdbcTemplate은 DataSource를 사용하는 복잡한 로직을 내부로 캡슐화했다. Service는 특히 DataSource를 직접적으로 사용하는 구조가 아니라 DB의 여러 작업을 하나의 트랜잭션으로 묶기 어렵다.

> UserService.java

```java
@Autowired
private Datasource dataSource;

public void upgradeLevels() {
		Connection connection = dataSource.getConnection();
		connection.setAutoCommit(false);
		...
		try {
				user.upgradeLevel();
				userDao.update(connection, user);
				connection.commit();
		} catch (IllegalArgumentException e) {
				connection.rollback();
		} finally {
				connection.close();
		}
}
```

* Data Access 관심사가 산재하여 Dao 및 Service를 분리했던 작업이 무용지물이 된다.
* UserDao 메서드의 시그니처가 변경된다.
  * 기존의 userDao는 jdbcTemplate을 통해 매번 새로운 Connection 객체를 만들며, 이는 매번 새로운 트랜잭션을 수행한다는 의미다.
  * 서비스에서 특정 Connection으로 트랜잭션 경계를 설정하더라도, 정작 update 등 메서드는 이와 무관한 트랜잭션 실행하게 된다.
    * userDao는 트랜잭션 선언에 사용된 Connection을 사용해야만 한다.
  * 더이상 UserDao 인터페이스는 데이터 기술에 독립적이지 않고 오히려 종속된다.
    * 메서드 시그니처에 Connection이 추가되지만 JPA를 사용하려면 Connection대신 EntityManager를 사용해야 한다.
    * 즉, 구현 기술이 바뀔 때마다 UserDao 인터페이스 및 서비스 코드가 변경되어야 한다.
    * 기존 테스트 코드들이 다 깨진다.
* 트랜잭션이 필요한 모든 메서드는 Connection 파라미터를 가져야 한다.
  * Spring Bean은 싱글톤이지만 매 트랜잭션마다 Connection이 달라지기 때문에, 별도의 인스턴스 필드로 저장해둘 수 없다.

### 2.2. 트랜잭션 동기화

트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고, 이후에 호출되는 Dao의 메소드에서는 저장된 Connection을 가져다가 사용하게 하는 것이다.

* 정확히는 Dao가 사용하는 JdbcTemplate이 트랜잭션 동기화 방식을 이용하도록 하는 것이다
* 트랜잭션 내 작업들이 완료되지 않으면 Connection을 계속 열어두며, 트랜잭션 내 Dao 메서드들은 트랜잭션을 위한 Connection을 계속 사용한다.
  * 커밋이나 롤백이 발생했을 시 Connection을 닫는다.
* 트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 오브젝트를 저장 하고 관리하기 때문에, 다중 사용자를 처리하는 서버의 멀티스레드 환경에서도 충돌이 나지 않는다.

> UserService.java

```java
public void upgradeLevels() throws Exception {
    TransactionSynchronizationManager.initSynchronization();
    Connection c = DataSourceUtils.getConnection(dataSource);
    c.setAutoCommit(false);
    try {
        List<User> users = userDao.findAll();
        for (User user : users) {
            if (canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
        c.commit();
    } catch (Exception e) {
        c.rollback();
    } finally {
        DataSourceUtils.releaseConnection(c, dataSource);
        TransactionSynchronizationManager.unbindResource(this.dataSource);
        TransactionSynchronizationManager.clearSynchronization();
    }
}
```

* Spring이 제공하는 트랜잭션 동기화 유틸 TransactionSynchronizationManaer 클래스를 사용한다.
* DataUtils로 Connection을 만들면 간편하면서 동시에 동기화에 사용되도록 저장소에 바인딩해준다.
* 이후 JdbcTemplate 작업들은 동기화한 Connection을 사용함으로써 같은 트랜잭션에 참여한다.
* JdbcTemplate은 미리 생성되어 트랜잭션 동기화 저장소에 등록된 DB Connection이나 트랜잭션이 없는 경우, 직접 DB Connection을 만들고 트랜잭션을 시작해서 작업을 진행한다.
  * 트랜잭션 동기화를 시작해 놓았다면 그 때부터 실행되는 JdbcTemplate의 메서드는 DB Connection을 직접 만드는 대신, 동기화 저장소에 있는 Connection을 사용한다.
* 기존의 트랜잭션 처리 방식과 다르게 파라미터에 Connection을 줄 필요가 없어서 인터페이스가 기술 종속적이지 않는다.

### 2.3. 트랜잭션 서비스 추상화

현재 위의 방식은 로컬 트랜잭션으로서 1개의 DB만 사용한다. 로컬 트랜잭션은 하나의 DB Connection에 종속적이기 때문에, 여러 DB에 대한 쓰기 작업들을 하나의 트랜잭션으로 묶을 수 없다. 이러한 경우 각 DB와 독립적으로 만들어지는 Connection을 통하는 것이 아닌, 별도의 트랜잭션 관리자를 통해 관리하는 글로벌 트랜잭션이 필요하다. 이를 통해 여러 개의 DB가 참여할 수 있는 트랜잭션을 생성할 수 있으며, JMS와 같은 트랜잭션 기능을 지원하는 서비스도 트랜잭션에 참여가 가능하다.

![image](https://user-images.githubusercontent.com/56240505/120425330-ef33bf00-c3a8-11eb-8ff0-ea48e718c17a.png)

* Java는 글로벌 트랜잭션을 위한 트랜잭션 매니저 지원 API인 JTA를 제공한다.
* 어플리케이션은 기존의 방법대로 DB는 JDBC, 메시징 서버는 JMS 같은 API를 사용해 필요한 작업을 수행한다.
* 트랜잭션은 JDBC나 JMS API가 아닌 JTA를 통해 트랜잭션 매니저가 관리하도록 위임한다.
  * 트랜잭션 매니저는 DB와 메시징 서버를 제어 및 관리하는 각각의 리소스 매니저와 XA 프로토콜을 통해 연결된다.
  * 이를 통해 트랜잭션 매니저가 실제 DB와 메시징 서버의 트랜잭션을 종합 제어가 가능하다.

> UserService.java

```java
public void upgradeLevels() throws Exception {
    InitialContext ctx = new InitialContext();
    UserTransaction tx = (UserTransaction)ctx.lookup(USER_TX_JNDI_NAME);
    tx.begin();
    Connection c = dataSource.getConnection();
    try {
        // 데이터 액세스 코드
        tx.commit();
    } catch (Exception e) {
        tx.rollback();
        throw e;
    } finally {
        c.close();
    }
}
```

* 로컬 트랜잭션을 사용하면 충분한 트랜잭션은 Jdbc를, 글로벌의 경우 JTA를 사용한 코드를 각각 사용해야 한다.
  * 특히 하이버네이트는 독자적인 트랜잭션 관리를 위해 Connection이 아닌 세션을 사용한다.
* 즉, UserService는 기술 환경에 따라 트랜잭션 경계 설정 코드가 변경되는 문제가 발생하며, 이는 특정 데이터 액세스 기술에 종속을 야기한다.

![image](https://user-images.githubusercontent.com/56240505/120426827-c5c86280-c3ab-11eb-99ac-dee7fcb0c9ce.png)

* 특정 기술에 종속되지 않도록 추상화를 적용한다.
  * 경계 설정 코드는 비슷한 패턴의 구조가 반복되기 때문에, 하위 시스템들의 공통점을 뽑아내면 하위 시스템이 변경되어도 일관되게 사용이 가능하다.
  * 다양한 DB 벤더사들이 제공하는 서로 다른 데이터 액세스 기술을 SQL이라는 공통점으로 추상한 것이 Jdbc다.

> UserService.java

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void upgradeLevels() {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        List<User> users = userDao.getAll();
        for (User user : users) {
            if (canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
        this.transactionManager.commit(status);
    } catch (RuntimeException e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}
```

* 스프링이 제공하는 트랜잭션 경계 설정을 위한 추상 인터페이스는 PlatformTransactionManager다.
  * JDBC의 로컬 트랜잭션은 PlatformTransactionManager의 구현체로 DataSourceTransactionManager를 사용한다.
  * JdbcTemplate에서 사용될 수 있는 방식으로 트랜잭션을 관리해준다.
* 글로벌 트랜잭션은 JTATransactionManager 구현체로 변경해주면 된다.
  * 즉, DI만 해주면 특정 데이터 액세스 기술에 종속되는 문제가 해결된다.
* Spring이 제공하는 모든 PlatformTransactionManager의 구현 클래스는 싱글톤 Bean으로 사용이 가능하다.
  * JtaTransactionManager는 애플리케이션 서버의 트랜잭션 서비스를 이용하기 때문에 직접 DataSource와 연동할 필요는 없다.
  * 대신 JTA를 사용하는 경우는 DataSource도 서버가 제공해주는 것을 사용해야 한다.

<br>

## 3. 서비스 추상화와 단일 책임 원칙

![image](https://user-images.githubusercontent.com/56240505/120428235-45efc780-c3ae-11eb-9725-f5625c0bf19a.png)

* UserDao와 UserService는 담당하는 코드의 기능적인 관심에 따라 분리되고, 서로 불필요한 영향을 주지 않으면서 독자적으로 확장이 가능하다.
  * 같은 애플리케이션 로직(계층)을 담은 코드지만 내용에 따라 수평적으로 분리되었다.
* 트랜잭션의 추상화는 애플리케이션의 비즈니스 로직과 그 하위에서 동작하는 로우 레벨의 트랜잭션 기술이라는 서로 다른 계층의 특성을 갖는 코드를 분리한 작업이다.
* 서로 영향을 주지 않고 자유롭게 확장될 수 있는 결합도가 낮은 구조를 만드는데 DI가 핵심이다.
  * 인터페이스 사용을 통해 구현체가 무엇인지 신경쓰지 않으며, DI는 관심, 책임, 성격이 다른 코드를 깔끔하게 분리한다.

### 3.1. 단일 책임 원칙

단일 책임 원칙은 하나의 모듈은 한 가지 책임을 가져야 한다는 의미다. 하나의 모듈이 바뀌는 이유는 한 가지여야 한다.

* 트랜잭션 추상화를 하지 않은 UserService는 사용자 레벨 관리와 트랜잭션 관리라는 두 가지 책임을 가진다.
  * 사용자 관리 로직이 변경되지 않더라도, 서버 환경 변화로 인해 Jdbc에서 JTA로 트랜잭션 기술이 변경되면 서비스 코드 수정이 수반된다.
  * 변경의 이유가 두 가지이며, 단일 책임 원칙을 위반한 사례다.
* 반면 트랜잭션 추상화 인터페이스를 사용하면, 사용자 관리 로직이 변경되지 않는한 서비스 코드는 수정할 이유가 없다.
  * 트랜잭션 기술 및 서버 환경이 바뀌더라도, 사용하는 UserDao 및 PlatformTransactionManager의 구현체만 변경하면 될 뿐이다.
* 장점
  * 변경이 발생할 때 수정할 대상이 명확해진다.
  * 기술이 변경되면 기술 계층과의 연동을 담당하는 기술 추상화 계층의 설정(구현체)만 바꾸면 된다.
    * 여러 클래스가 구체를 통해 서로 강결합한다면, 한 쪽의 변경이 다른 한 쪽에 영향을 전파한다.
    * 특정 기술에 종속되기 때문에, 기술 교체시 프로젝트 전반의 코드 수정이 빈번해진다.

### 3.2. 핵심

* 책임과 관심이 다른 코드를 적절하게 분리하고, 서로 영향을 주지 않도록 추상화 기법을 도입한다.
  * 어플리케이션 로직과 기술 및 환경을 분리하는 작업은 복잡한 엔터프라이즈 어플리케이션에서 필수다.

> UserService.java

```java
public void upgradeLevels() {
    PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
    //로직 수행
}
```

* 추상화를 하더라도 인터페이스 구체 클래스를 특정 클래스 내부에서 직접 생성하는 것은 기술 변경시 코드 수정을 의미한다.

> UserService.java

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void upgradeLevels() {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    //로직 수행
}
```

* Spring이 제공하는 DI는 사용하는 인터페이스의 구체 클래스 생성 및 주입을 담당하며, 이는 코드 사이 결합을 최소화한다.
* 단일 책임 원칙을 잘 지키는 코드를 위해 인터페이스를 도입하고 이를 DI로 연결해야 한다.
  * 그 결과로 단일 책임 원칙뿐 아니라 개방 폐쇄 원칙도 잘 지키고, 모듈 간에 결합도가 낮아서 서로의 변경이 영향을 주지 않는다.
  * 같은 이유로 변경이 단일 책임에 집중되는 응집도 높은 코드가 된다.
  * DI는 테스트 환경 구축에도 용이하다.

<br>

## 4. 메일 서비스 추상화

![image](https://user-images.githubusercontent.com/56240505/120429767-ecd56300-c3b0-11eb-9f81-7285474573b9.png)

* 테스트 할 때 실제 운영 메일 서버를 사용하지 않는다.
  * 테스트 진행할 때 마다 실제로 메일을 발송하면 운영 서버 부하가 커진다.
* 테스트용 메일 서버를 사용할 수 있으나, 실제 수신 확인에 대한 테스트 필요성에 대해 의문을 가져야 한다.
  * 메일 서버는 충분히 테스트된 시스템이며, SMTP 프로토콜로 전송 요청을 받으면 메일이 잘 전송됐다고 신뢰해도 충분하다.
  * 즉, JavaMail을 통해 메일 서버까지만 메일이 잘 전달되면, 클라이언트가 잘 받았다고 판단해도 좋다.
* 따라서 테스트용 메일 서버는 외부로 메일을 발송하지 않고 요청만 받아도 충분하다.

![image](https://user-images.githubusercontent.com/56240505/120430370-e0053f00-c3b1-11eb-966f-9206637b8d1b.png)

* 비슷하게 JavaMail은 표준 기술이며 수 많은 시스템에 사용돼서 검증된 안정적인 모듈이다.
  * 따라서 JavaMail API를 통해 요청이 들어간다는 보장만 있으면 JavaMail을 구동시킬 필요가 없다.
  * JavaMail이 동작하면 외부 메일 서버 및 네트워크 연동으로 인해 부하가 커지기 때문이다.
* 매번 검증이 불필요한 메일 전송 요청을 보내지 않으면 테스트 성능이 향상된다.

### 4.1. Java Mail의 문제점

Java Mail은 JdbcTemplate처럼 DataSource를 갈아 끼움으로써 테스트용 객체를 만들 수 있는 구조가 아니다. 메일을 보낼 때 사용하는 ``Session s = Session.getInstance(props, null);`` 코드 예제처럼, Session은 인터페이스가 아닌 상속 불가능한 final 클래스이며 생성자도 정적 팩토리만 제공한다. 즉, 테스트하기 어렵다.

### 4.2. Java Mail 추상화

> MailSender.java

```java
package org.springframework.mail; ...

public interface MailSender {
		void send(SimpleMailMessage simpleMessage) throws MailException;
		void send(SimpleMailMessage[] simpleMessages) throws MailException;
}
```

> UserService.java

```java
@Autowired
private MailSender mailSender;

private void sendEmail(User user) {
    mailSender.setHost("mail.server.com");
    SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
    simpleMailMessage.setTo(user.getEmail());
    simpleMailMessage.setFrom("useradmin@ksug.org");
    simpleMailMessage.setSubject("Upgrade 안내");
    simpleMailMessage.setText("사용자님의 등급이 " + user.getLevel().name());
    mailSender.send(simpleMailMessage);
}
```

* 다행히 Spring은 Java Mail에 대한 추상화 기능을 제공한다.
  * 기존 Java Mail 사용시 발생하는 확인 예외들을 런타임 예외로 변환한다.
* MailSender의 구현체로 Spring이 제공하는 JavaMailSenderImpl 오브젝트를 주입한다.
  * 내부적으로 Java Mail API를 사용해 메일을 전송한다.
  * 구현체를 갈아 끼운다면 테스트용 Mock 객체를 만들 수 있다.

### 4.3. 테스트와 서비스 추상화

![image](https://user-images.githubusercontent.com/56240505/120431938-25c30700-c3b4-11eb-8252-acb957fe172f.png)

> DummyMailSender.java

```java
public class DummyMailSender implements MailSender {

    public void send(SimpleMailMessage mailMessage) throws MailException { }
    public void send(SimpleMailMessage[] mailMessage) throws MailException { }
}
```

* Spring에서 제공하는 MailSender 인터페이스의 구현 클래스는 JavaMailSenderImpl 하나 뿐이다.
  * 그러나 이러한 추상화를 통해 다른 메시징 서버의 API를 사용하는 구현 클래스 및 테스트용 구현 클래스 DI가 가능하다는 장점이 존재한다.
* DummyMailSender는 아무런 작업을 하지 않지만, 기존 테스트들이 메일 전송 관련 예외가 발생하지 않고 실행되게 해주는 역할을 맡을 뿐이다.
* 메일 발송 기능에도 트랜잭션 개념을 적용해야 한다.
  * 사용자 계급 관리 기능이 롤백됬는데, 승급했다는 메일이 전송된다면 오류다.
  * MailSender를 구현한 트랜잭션 기능을 가지고 있는 메일 전송용 구현 클래스를 만들어준다.
    * 확장이 불가능한 JavaMail에 비해, 변화하는 기술 및 환경에 유연하게 대처할 수 있다.

### 4.4. 테스트 대역

![image](https://user-images.githubusercontent.com/56240505/120433004-b3ebbd00-c3b5-11eb-81e2-520d0050b295.png)

* UserServiceTest의 관심사는 UserService에서 구현한 사용자 정보 가공 로직일 뿐, 메일이 어떻게 전송될 것이 아니다.
* 테스트를 위해 번거롭게 메일 발송 코드를 삭제했다가 복구하는 작업은 현실적으로 불가능하다.
  * 테스트 대상 코드를 수정하지 않고, UserService 자체에 대한 테스트 지장을 주지 않기 위해 Mock 객체를 사용한 것이다.
* UserDaoTest도 UserDao의 동작에 관심있을 뿐, 그 이면의 DB Connection 풀 등에는 관심이 없다.
  * 따라서 DataSource를 DI할 수 있는 구조를 사용하여, 운영 DB와는 별개로 테스트 환경에서 간편하고 신속하게 동작할 수 있는 간단한 DataSource를 사용하도록 한다.
* 이렇듯, 실전에서는 사용할 객체를 교체할 가능성이 없더라도 테스트만을 위해서도 DI는 유용하다.
* 테스트 대역이란?
  * 테스트 대상이 사용하는 의존 오브젝트를 대체할 수 있도록 만든 오브젝트다.
  * 테스트용으로 사용되는 특별한 오브젝트로서, 대부분 테스트 대상인 오브젝트의 의존 오브젝트들이다.
  * 테스트 환경을 만들어주기 위해, 테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 빠르게, 자주 테스트를 실행할 수 있도록 사용하는 오브젝트다.
    * 테스트 용도로 주입되는 SimpleConnectionDataSource, DummyMailSender 등이다.

### 4.5. 테스트 스텁

* 테스트 스텁이란 테스트 대상 오브젝트의 의존객체로서 존재하면서, 테스트 동안에 코드가 원활하게 동작하도록 돕는 것이다.
  * 따라서 테스트 수행 전에 미리 의존 오브젝트를 스텁으로 DI 해야 한다.
* 테스트 코드 내부에서 간접적으로 사용되며, DummyMailSender이 대표적인 테스트 스텁의 예시다.
* 테스트 스텁은 아무런 동작을 안 할수도, 혹은 결과를 반환하거나 예외를 호출할 수도 있다.
  * 그래야 다양한 상황에서 테스트 대상 객체가 어떻게 반응하는지 확인할 수 있기 때문이다.
* 스텁을 이용하면 간접적인 입력 값을 지정해줄 수 있으며, 혹은 간접적인 출력 값을 받게 할 수 있다.

```java
@InjectMocks
private LineService lineService;

@Mock
private LineRepository lineRepository;

@Mock
private SectionService sectionService;

@Test
void createLine() {
    //... 생략
    given(lineRepository.save(line)).willReturn(lineId);
    given(sectionService.createSection(lineRequest, lineId)).willReturn(lineId);
    given(lineRepository.findById(lineId)).willReturn(retrievedLine);

    Line savedLine = lineService.createLine(lineRequest);

    assertThat(savedLine).isEqualTo(retrievedLine);
    verify(lineRepository, times(1)).save(line);
    verify(lineRepository, times(1)).findById(lineId);
    verify(sectionService, times(1)).createSection(lineRequest, lineId);
}
```

* 테스트 오브젝트가 간접적으로 의존 오브젝트에 넘기는 값과 그 행위 자체에 대해서도 검증하기 위해서는 **Mock 객체**를 사용해야 한다.
  * Mock 객체는 테스트 대상 객체와 자신의 사이에서 발생하는 커뮤니케이션 내용을 저장해뒀다가, 검증에 활용할 수 있게 해준다.
* 보통의 테스트로는 검증하기가 어려운 테스트 대상 오브젝트의 내부에서 일어나는 다른 의존 오브젝트 사이에서 주고받는 정보까지 검증 가능함이 장점이다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
