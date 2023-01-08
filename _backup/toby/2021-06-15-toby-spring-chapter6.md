---
title: "[토비의 스프링] 6장. AOP"
excerpt: "AOP는 OOP를 보완하는 기법이다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
date: 2021-06-15
last_modified_at: 2021-06-15
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter6)

## 1. 트랜잭션 코드의 분리

> UserService.java

```java
public void upgradeLevels() {
    TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        userDao.getAll()
                .forEach(this::upgradeLevel);
        transactionManager.commit(transaction);
    } catch (Exception exception) {
        transactionManager.rollback(transaction);
        throw exception;
    }
}
```

* 트랜잭션 경계 설정 코드와 비즈니스 로직 코드가 산재해있다.
  * 두 코드는 서로 정보를 교환하지 않고 완벽히 상호 독립적인 상태다.

### 1.1. 인터페이스 추출

> UserService.java

```java
public interface UserService {

    void upgradeLevels();

    void add(User user);
}
```

> UserServiceImpl.java

```java
@Service
public class UserServiceImpl implements UserService {

    private UserDao userDao;
    private UserLevelUpgradePolicy userLevelUpgradePolicy;
    private MailSender mailSender;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void setUserLevelUpgradePolicy(UserLevelUpgradePolicy userLevelUpgradePolicy) {
        this.userLevelUpgradePolicy = userLevelUpgradePolicy;
    }

    public void setMailSender(MailSender mailSender) {
        this.mailSender = mailSender;
    }

    @Override
    public void upgradeLevels() {
        userDao.getAll()
                .forEach(this::upgradeLevel);
    }

    protected void upgradeLevel(User user) {
        if (userLevelUpgradePolicy.canUpgradeLevel(user)) {
            userLevelUpgradePolicy.upgradeLevel(user);
            userDao.update(user);
            sendEmail(user);
        }
    }

    private void sendEmail(User user) {
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        simpleMailMessage.setTo(user.getEmail());
        simpleMailMessage.setFrom("useradmin@ksug.org");
        simpleMailMessage.setSubject("Upgrade 안내");
        simpleMailMessage.setText("사용자님의 등급이 " + user.getLevel().name());
        mailSender.send(simpleMailMessage);
    }

    @Override
    public void add(User user) {
        if (user.getLevel() == null) {
            user.setLevel(Level.BASIC);
        }
        userDao.addUser(user);
    }
}
```

> UserServiceTx.java

```java
public class UserServiceTx implements UserService {

    private UserService userService;
    private PlatformTransactionManager transactionManager;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    @Override
    public void upgradeLevels() {
        TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels();
            transactionManager.commit(transaction);
        } catch (RuntimeException e) {
            transactionManager.rollback(transaction);
            throw e;
        }
    }

    @Override
    public void add(User user) {
        userService.add(user);
    }
}
```

* UserService를 인터페이스로 추출한 뒤, 비즈니스 로직만을 가진 UserServiceImpl 구현체를 생성한다.
* UserServiceTx는 UserService 인터페이스를 구현하며, 주입받은 UserService의 메서드 호출 전후로 트랜잭션 경계 설정만을 담당한다.
  * 주입되는 UserService Bean의 구현체는 당연히 UserServiceImpl이 된다.
* ``Client <-> [UserService ---> UserServiceTx ---> UserServiceImpl]``의 구조다.

> applicationContext.xml

```xml
<bean id="userService" class="springbook.service.UserServiceTx">
    <property name="userService" ref="userServiceImpl"/>
    <property name="transactionManager" ref="transactionManager"/>
</bean>
<bean id="userServiceImpl" class="springbook.service.UserServiceImpl">
    <property name="userDao" ref="userDaoJdbc"/>
    <property name="userLevelUpgradePolicy" ref="userLevelUpgradePolicy"/>
    <property name="mailSender" ref="mailSender"/>
</bean>
```

> UserServiceTest.java

```java
@Autowired
private UserService userService;
```

* UserService 인터페이스에 해당하는 타입의 Bean이 두 개 존재한다.
* @Autowired는 하나의 Bean을 결정할 수 없는 경우에는 필드 이름을 이용해 Bean을 찾아 주입한다.
  * 위 설정의 경우 최종적으로 UserServiceTx 타입의 Bean이 주입된다.
  * 그 외의 경우 [@Primary나 @Qualifier 등의 옵션을 통해](https://xlffm3.github.io/spring%20&%20spring%20boot/Chapter3_Autowire/) 주입받을 Bean을 명시한다.

### 1.2. 장점

1. 비즈니스 로직을 담당하고 있는 UserServiceImpl의 코드를 작성할 때는 트랜잭션과 같은 기술적인 내용에는 전혀 신경 쓰지 않아도 된다.
  * UserServiceTx와 같은 트랜잭션 기능을 가진 오브젝트가 먼저 실행되도록 만들기만 하면 된다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.

<br>

## 2. 고립된 단위 테스트

* 가능한 한 작은 단위로 쪼개서 기능을 테스트하느 것이 좋다.
* 테스트가 실패했을 때 단위가 큰 경우 실행되는 코드의 양아 믾아 원인을 특정하기 힘들다.
* 테스트의 단위가 작아야 테스트 의도나 내용이 분명하며 만들기도 쉽다.

### 2.1. UserServiceTest의 문제점

![image](https://user-images.githubusercontent.com/56240505/122226668-28177c00-cef1-11eb-85f3-567b9f38217e.png)

* 현재 UserServiceTest는 UserDao, TransactionManager, MailSender 등 다양한 오브젝트와 의존 관계를 형성한다.
* UserService를 테스트하는데 이면에는 많은 리소스가 요구되며, 다양한 오브젝트 및 서비스에 의존한다.
  * DB 드라이버 세팅와 DB 서버와의 네트워크 연결 및 메일 서버 셋업 등.
  * 다양한 오브젝트와 환경, 서비스, 서버, 심지어 네트워크까지 함께 테스트되는 것이다.
  * 테스트를 준비하기 어렵고 테스트 수행 시간이 오래 걸리며, 환경이 살짝 달라지면 테스트가 깨지기 쉽다.
  * DAO 메서드를 수행하기 위해 여러 테이블을 조인해야하는 경우, 데이터 준비가 매우 복잡해진다.

### 2.2. 테스트 대상 오브젝트 고립

* 테스트의 대상이 환경이나, 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다.
* 테스트 스텁이나 목 오브젝트와 같은 테스트 대역을 사용한다.

> MockUserDao.java

```java
static class MockUserDao implements UserDao {

     private List<User> users;
     private List<User> updated = new ArrayList();

     private MockUserDao(List<User> users) {
         this.users = users;
     }

     public List<User> getUpdated() {
         return updated;
     }

     @Override
     public void addUser(User user) {
         throw new UnsupportedOperationException();
     }

     @Override
     public User getUser(String id) {
         return null;
     }

     @Override
     public void deleteAll() {
         throw new UnsupportedOperationException();
     }

     @Override
     public int getCount() {
         throw new UnsupportedOperationException();
     }

     @Override
     public List<User> getAll() {
         return this.users;
     }

     @Override
     public void update(User user1) {
         updated.add(user1);
     }
}
```

* UserDao는 단지 테스트 대상의 코드가 정상적으로 수행되도록 도와주기만 하는 스텁이 아니라, 부가적인 검증 기능까지 가진 목 오브젝트로 만든다.
* 고립된 환경에서 동작하는 ``upgradeLevels()``의 테스트 결과를 검증할 방법이 필요하기 때문이다.
  * 고립된 환경을 위해 실제 DB 커넥션에 의존하지 않지만 그 협력 오브젝트인 UserDao에게 어떤 요청을 했는지를 확인하는 작업이 필요하다.
  * 테스트 중에 DB에 결과가 반영되지는 않더라도, UserDao의 ``update()`` 메소드를 호출하는 것을 확인할 수 있다면?
    * DB에 그 결과가 반영될 것이라고 결론을 내릴 수 있다.
* ``getAll()``에 대해서는 스텁으로서, ``update()``에 대해서는 목 오브젝트로서 동작하는 UserDao 타입의 테스트 대역을 생성한다.
* 완벽하게 UserServiceTest를 고립시킨다면 ``@ExtendWith()``이나 Spring Context를 사용하지 않고도 테스트를 진행할 수 있다.
  * 테스트가 다른 의존 대상에 영향을 받을 경우를 대비해 복잡하게 준비할 필요가 없다.
  * 테스트 수행 성능도 크게 향상된다.

### 2.3. 단위 테스트와 통합 테스트

* 단위 테스트의 단위란 정의하기 나름이다.
* 이 책에서는 ``테스트 대상 클래스를 목 오브젝트 등의 테스트 대역을 이용해 의존 오브젝트나 외부의 리소스를 사용하지 않도록 고립시켜서 테스트하는 것``을 단위 테스트라 정의한다.
  * 반면 외부의 DB나 파일 및 서비스 등의 리소스가 참여하는 테스트는 통합 테스트라고 칭한다.
* 여러 개의 단위가 의존관계를 가지고 동작할 때를 위한 통합 테스트는 필요하다.
  * 다만, 단위 테스트를 충분히 거쳤다면 통합 테스트의 부담은 상대적으로 줄어든다.
* 항상 단위 테스트를 먼져 고려하자.
* Spring의 테스트 컨텍스트 프레임워크를 이용하는 테스트는 통합 테스트다.

### 2.4. Mockito Framework

단위 테스트 작성시 목 오브젝트를 만드는 일이 매우 번거롭다. MockUserDao처럼 사용하지 않는 인터페이스를 일일이 구현해야 한다. Mockito Framework는 목 클래스를 일일이 준비하지 않고, 간단한 메소드 호출로 특정 인터페이스를 구현한 테스트용 목 오브젝트를 생성할 수 있다.

* @InjectMocks와 @Mock 애너테이션에 대한 글은 [여기](https://xlffm3.github.io/java/Mockito/)를 참고하자.

<br>

## 3. Dynamic Proxy & Factory Bean

![image](https://user-images.githubusercontent.com/56240505/122242588-67989500-cefe-11eb-8514-34c356e69fd8.png)

* 현재 클라이언트는 UserService 인터페이스를 통해 서비스 로직을 수행한다.
  * 실제로는 트랜잭션 관리 부가 기능을 가진 UserSerivceTx 구현체가 동작한다.
  * UserServiceTx 구현체는 필드로 주입받은 내부 UserService 인터페이스(UserServiceImpl 구현체)를 통해 실제 비즈니스 로직을 호출한다.
* 핵심 비즈니스 로직을 가진 구현체는 부가 기능을 가진 구현체를 알지 못한다.
* 클라이언트가 핵심 기능을 가진 클래스를 직접 사용해버리면 부가기능이 적용될 기회가 없다.
  * 따라서 부가기능은 마치 자신이 핵심 기능을 가진 클래스인 것처럼 꾸며서, 클라이언트가 자신을 거쳐서 핵심기능을 사용하도록 한다.
  * 부가 기능이 핵심기능을 사용하는 구조가 되는 것이다.
* Proxy : 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받는다.
* Target or Real Subject : 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트다.

### 3.1. Proxy 특징

* 타깃과 같은 인터페이스를 구현했으며, 프록시가 타깃을 제어할 수 있는 위치에 있다.
* 사용 목적에 따라 두 가지로 구분이 가능하다.
  * 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서 사용한다.
  * 타깃에 부가적인 기능을 부여하기 위해 사용한다.

### 3.2. 데코레이터 패턴

* 타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴을 말한다.
  * 컴파일 시점, 즉 코드 상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져 있지 않다는 의미다.
* 데코레이터 패턴에서는 프록시가 한 개로 제한되지 않고, 여러 개의 데코레이터 프록시를 사용할 수 있다.
  * 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근한다.
  * 자신이 최종 타깃으로 위임하는지, 다음 단게의 데코레이터 프록시에게 위임하는지 알지 못한다.
    * ``InputStream is = new BufferedInputStream(new FileInputStream("a.txt"));``

> applicationContext.xml

```xml
<bean id="userService" class="springbook.service.UserServiceTx">
    <property name="userService" ref="userServiceImpl"/>      
    <property name="transactionManager" ref="transactionManager"/>
</bean>
<bean id="userServiceImpl" class="springbook.service.UserServiceImpl">
    <property name="userDao" ref="userDaoJdbc"/>
    <property name="userLevelUpgradePolicy" ref="userLevelUpgradePolicy"/>      
    <property name="mailSender" ref="mailSender"/>
</bean>
```

* UserServiceTx로 선언된 userService Bean은 데코레이터 프록시다.
* Spring DI를 통해 데코레이터 패턴을 런타임시 다이내믹하기 설정할 수 있다.
* 실제 UserServiceTx 내부에서는 UserServiceImpl이라는 구현체가 아닌 UserService 인터페이스를 통해 작업을 위임한다.
  * 따라서 UserService 인터페이스를 구현한 데코레이터를 언제든 만들어서 UserServiceTx와 UserServiceImpl 사이에 추가할 수 있다.

### 3.3. 프록시 패턴

* 프록시 패턴은 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우에 사용한다.
* 프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다.
  * 대신 클라이언트가 타깃에 접근하는 방식을 변경해준다.

#### 예시

* 타깃 오브젝트 생성이 복잡하거나 당장 필요하지 않은 경우 생성하지는 않는다.
  * 타킷 오브젝트에 대한 레퍼런스가 필요한 경우 프록시를 넘겨준다.
  * 프록시의 메서드를 통해 타깃을 시도하려는 순간, 즉 꼭 필요한 순간이 생길 때 프록시가 타깃 오브젝트를 생성하고 요청을 위임한다.
  * 프록시를 통한 생성 지연을 통해 얻는 이점이 많다.
* EJB 등 각종 리모팅 기술을 통해 다른 서버에 존재하는 원격 오브젝트를 사용하는 경우, 프록시를 만들면 클라이언트는 로컬에 존재하는 오브젝트를 사용하는 것처럼 프록시를 사용할 수 있다.
* 특별한 상황에서 타깃에 대한 접근 권한을 제어한다.
  * Collections의 ``unmodifiableCollection()``을 통해 반환된 컬렉션 오브젝트는 쓰기 작업을 제어하는 프록시 오브젝트다.

### 3.4. 프록시 작성의 문제점과 해결책

1. 타깃의 인터페이스 구현하고 위임하는 코드 작성이 번거롭다.
  * 부가 기능이 필요없는 메서들도 위임을 위해 일일이 작성해야 한다.
2. 부가 기능의 코드가 중복될 공산이 크다.
  * 프록시의 여러 위임 메서드에서 공통적으로 트랜잭션 경계를 설정하는 코드가 발생할 수 있다.

#### Reflection

> HelloTarget.java

```java
public class HelloTarget implements Hello {

    @Override
    public String sayHello(String name) {
        return "Hello" + name;
    }

    @Override
    public String sayHi(String name) {
        return "Hi" + name;
    }

    @Override
    public String sayThankYou(String name) {
        return "Thank You" + name;
    }
}
```

> Uppercasehandler.java

```java
public class Uppercasehandler implements InvocationHandler {

    private Hello target;

    public Uppercasehandler(Hello target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String result = (String) method.invoke(target, args);
        return result.toUpperCase();
    }
}
```

> HelloTest.java

```java
@Test
void proxyWithReflection() {
    Hello proxy = (Hello) Proxy.newProxyInstance(getClass().getClassLoader(), new Class[]{Hello.class}, new Uppercasehandler(new HelloTarget()));

    assertThat(proxy.sayHello("abc")).isEqualTo("HELLOABC");
    assertThat(proxy.sayHi("abc")).isEqualTo("HIABC");
    assertThat(proxy.sayThankYou("abc")).isEqualTo("THANK YOUABC");
}
```

![image](https://user-images.githubusercontent.com/56240505/122375827-8babb300-cf9e-11eb-9536-500af9b6fa30.png)

* Java의 모든 클래스는 그 클래스 자체의 구성 정보를 가진 Class 타입의 오브젝트를 가진다.
  * ``getClass()``를 통해 얻을 수 있으며, 이를 통해 클래스 코드에 대한 메타 정보를 가져오거나 오브젝트 조작이 가능하다.
  * 메서드나 인터페이스 확인이 가능하며, 심지어 필드까지 수정할 수 있다.
* Reflection을 활용하면 타깃의 인터페이스를 일일이 구현할 필요가 없다.
  * 또한 타깃의 메서드 요청이 InvocationHandler의 ``invoke()`` 메서드로 집중되기에 중복이 사라진다.
* 다이내믹 프록시는 런타임 코드 자동 생성 기법이다.
  * JDK의 다이내믹 프록시 기술은 특정 인터페이스를 구현한 오브젝트에 대해서 프록시 역할을 해주는 클래스를 런타임시 내부적으로 만들어준다.
  * 즉, 프록시 클래스 없이도 프록시 오브젝트를 런타임 시에 만들어준다.

> Uppercasehandler.java

```java
public class Uppercasehandler implements InvocationHandler {

    private Object target;

    public Uppercasehandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object object = method.invoke(target, args);
        if (object instanceof String && method.getName().startsWith("say")) {
            return ((String) object).toUpperCase();
        }
        return object;
    }
}
```

* Reflection을 사용하는 경우, 타깃을 특정 타입으로 제한할 필요가 없다.

> TransactionHandler.java

```java
public class TransactionHandler implements InvocationHandler {

    private Object target;
    private PlatformTransactionManager transactionManager;
    private String pattern;

    public void setTarget(Object target) {
        this.target = target;
    }

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public void setPattern(String pattern) {
        this.pattern = pattern;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().startsWith(pattern)) {
            return invokeInTransaction(method, args);
        }
        return method.invoke(target, args);
    }

    private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
        TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object result = method.invoke(target, args);
            transactionManager.commit(transaction);
            return result;
        } catch (InvocationTargetException e) {
            transactionManager.rollback(transaction);
            throw e.getTargetException();
        }
    }
}
```

* UserServiceTx라는 데코레이터 프록시를 Reflection을 활용하여 다이내믹 프록시로 생성할 수 있다.

### 3.5. 다이내믹 프록시의 문제점

* Spring은 내부적으로 리플렉션 API를 이용해 Bean 정의의 클래스 이름을 가지고 Bean 오브젝트를 생성한다.
  * 이후 정의된 프로퍼티 값을 주입하는 방식이다.
* DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 Spring Bean으로 등록할 방법이 없다.
  * 다이내믹 프록시 오브젝트는 ``newProxyInstance()``를 사용해서만 생성할 수 있다.
* Factory Bean은 Spring을 대신해 오브젝트의 생성 로직을 담당하는 Bean이며, 이를 통해 문제를 해결한다.

> MessageFactoryBean.java

```java
public class MessageFactoryBean implements FactoryBean<Message> {

    private String text;

    public void setText(String text) {
        this.text = text;
    }

    @Override
    public Message getObject() throws Exception {
        return Message.newMessage(text);
    }

    @Override
    public Class<?> getObjectType() {
        return Message.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

> applicationContext.xml

```xml
<bean id="message" class="springbook.factory.MessageFactoryBean">
    <property name="text" value="Factory Bean"/>
</bean>
```

* 등록하려는 Bean은 Message지만 클래스 어트리뷰트는 Spring의 FactoryBean 인터페이스를 구현하고 Factory로 등록한다.
* 다이내믹 프록시를 직접 Bean으로 등록할 수 없다면, Factory Bean 방식을 통해 등록해본다.

### 3.6. Factory Bean의 장점 및 한계

> TxFactoryBean.java

```java
public class TxFactoryBean implements FactoryBean<Object> {

    private Object target;
    private PlatformTransactionManager transactionManager;
    private String pattern;
    private Class<?> serviceInterface;

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    public void setPattern(String pattern) {
        this.pattern = pattern;
    }

    public void setServiceInterface(Class<?> serviceInterface) {
        this.serviceInterface = serviceInterface;
    }

    @Override
    public Object getObject() throws Exception {
        TransactionHandler transactionHandler = new TransactionHandler();
        transactionHandler.setTransactionManager(transactionManager);
        transactionHandler.setTarget(target);
        transactionHandler.setPattern(pattern);
        return Proxy.newProxyInstance(getClass().getClassLoader(),
                new Class[]{serviceInterface},
                transactionHandler);
    }

    public void setTarget(Object target) {
        this.target = target;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

> applicationContext.xml

```xml
<bean id="userService" class="springbook.factory.TxFactoryBean">
    <property name="pattern" value="upgradeLevels"/>
    <property name="target" ref="userServiceImpl"/>
    <property name="serviceInterface" value="springbook.service.UserService"/>
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```

* TxFactoryBean은 타입에 상관없이 재사용이 가능하다.
  * UserService가 아닌 다른 타입의 Bean을 이용하려면 비슷하게 XML 설정만 수정하면 된다.
* 프록시 팩토리 Bean을 사용하면 직접 프록시를 구현하는 것보다 장점이 많다.
  * 프록시 타겟 인터페이스의 메서드 구현을 할 필요가 없다.
  * 데코레이터 프록시의 여러 부가 기능 코드가 중복된다.
* 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 것은 불가능하다.
  * 현재는 하나의 클래스에 존재하는 여러 메서드들에 대해 공통적인 부가 기능만을 제공할 수 있다.
  * 결국 여러 클래스에 트랜잭션을 적용하려면 이와 같은 FactoryBean을 각 클래스별로 작성하는 중복이 발생한다.
* 하나의 타깃에 여러가지 부가 기능을 추가하기 힘든 구조다.
  * XML 설정이 매우 비대해진다.
* TransactionHandler가 프록시 FactoryBean의 개수 만큼 만들어져야 한다.
  * TxFactoryBean 내부가 아닌 Bean으로 등록될 수 있으나, 타깃 오브젝트가 다르기 때문에 타깃 오브젝트 개수만큼 등록해야 한다.

<br>

## 4. Spring의 Proxy Factory Bean

* Java는 JDK의 다이내믹 프록시 외에도 편리하게 프록시를 만드는 기능을 제공한다.
* Spring은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다.
   * 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공한다.

> HelloTest.java

```java
@Test
void proxyFactoryBean() {
    ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
    proxyFactoryBean.setTarget(new HelloTarget());
    proxyFactoryBean.addAdvice(new UppercaseAdvice());

    Hello proxy = (Hello) proxyFactoryBean.getObject();
    assertThat(proxy.sayHello("abc")).isEqualTo("HELLOABC");
    assertThat(proxy.sayHi("abc")).isEqualTo("HIABC");
    assertThat(proxy.sayThankYou("abc")).isEqualTo("THANK YOUABC");
}

static class UppercaseAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        String ret = (String) invocation.proceed();
        return ret.toUpperCase();
    }
}
```

* ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고, 프록시를 통해 제공해줄 부가기능은 별도의 Bean에 둘 수 있다.
* 부가 기능은 MethodInterceptor 인터페이스를 구현해서 만든다.
  * InvocationHandler의 ``invoke()`` 메서드는 타깃 오브 젝트에 대한 정보를 제공하지 않는다.
  * 따라서 InvocationHandler 구현체는 타깃에 대한 정보를 직접 알아야한다.
  * 반면 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다.
    * ProxyFactoryBean으로부터 메서드와 타깃 오브젝트의 정보인 MethodInvocation을 제공받기 때문이다.
    * **타깃이 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 Bean으로 등록 가능하다.**

### 4.1. Advice : 타깃이 필요없는 순수 부가 기능

* ``proceed()``는 타깃 오브젝트의 메서드를 실행해주는 기능이 있다.
  * 따라서 MethodInvocation은 일종의 콜백 오브젝트이며, MethodInteceptor는 일종의 템플릿처럼 공유될 수 있다.
* ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용했기에 템플릿 역할을 하는 MethodInterceptor을 싱글톤 공유가 가능하다.
  * JdbcTemplate 또한 특정 SQL 파라미터에 종속되지 않아 수 많은 DAO가 공유할 수 있다.
* ProxyFactoryBean 하나로도 ``addAdvice()``를 통해 여러 개의 부가 기능올 추가한 프록시를 만들 수 있다.
* Advice는 타깃 오브젝트에 종속되지 않는 순수한 부가기능을 담은 오브젝트다.
  * MethodInterceptor는 Advice 인터페이스를 확장했다.
* ProxyFactoryBean은 인터페이스 타입을 제공받지 않더라도 자동 검출 기능을 통해 구현한 인터페이스를 알아낸다.

### 4.2. Pointcut : 부가기능 적용 대상 메소드 선정 방법

* 기존의 TxFactoryBean은 내부적으로 프록시 객체를 생성할 때 InvocationHandler를 생성하고, 어떤 타깃과 어떤 메서드에 적용할지 등을 설정한다.
  * 타깃과 메서드가 다르다면 InvocationHandler 오브젝트를 여러 프록시가 공유할 수 없다.

![image](https://user-images.githubusercontent.com/56240505/122414233-f1f5fd00-cfc1-11eb-97fe-aa097f042287.png)

* Spring의 ProxyFactoryBean 방식은 부가 기능 오브젝트(Advice)와 메서드 선정 알고리즘 오브젝트(Pointcut)을 통해 확장 가능하고 유연한 구조를 제공한다.
  * 두 가지 모두 프록시에 DI되며, 여러 프록시에서 공유될 수 있어 싱글톤 Bean으로 등록할 수 있다.
* 프록시는 클라이언트로부터 요청을 받으면 포인트컷에게 부가 기능을 부여할 메서드인지를 확인 요청한다.
* 프록시는 포인트컷으로부터 부가 기능을 적용할 대상 메소드인지 확인받으면, MethodInteceptor 타입의 어드바이스를 호출한다.
  * 타깃 메서드의 호출이 필요하면 MethodInvocation 타입 콜백 오브젝트의 ``proceed()`` 메서드를 호출한다.
* 재사용 가능한 기능을 만들어두고 바뀌는 부분(콜백 오브젝트와 메소드 호출정보)만 외부에서 주입해서 이를 작업 흐름(부가기능 부여) 중에 사용하도록 하는 전형적인 템플릿/콜백 구조다.
  * 프록시와 ProxyFactoryBean 등의 변경 없이도 기능을 자유롭게 확장할 수 있는 OCP가 충족된다.

> HelloTest.java

```java
@Test
void pointcutAdvisor() {
    ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
    proxyFactoryBean.setTarget(new HelloTarget());

    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("sayH*");
    proxyFactoryBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));

    Hello proxy = (Hello) proxyFactoryBean.getObject();
    assertThat(proxy.sayHello("abc")).isEqualTo("HELLOABC");
    assertThat(proxy.sayHi("abc")).isEqualTo("HIABC");
    assertThat(proxy.sayThankYou("abc")).isNotEqualTo("THANK YOUABC");
}
```

* 여러 개의 어드바이스와 포인트컷이 추가될 수 있다.
* 어느 어드바이스에 대한 포인트컷인지 애매해지기 때문에 이 둘을 묶은 Advisor 오브젝트를 등록한다.

> TransactionAdvice.java

```java
public class TransactionAdvice implements MethodInterceptor {

    private PlatformTransactionManager transactionManager;

    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        TransactionStatus transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object result = invocation.proceed();
            transactionManager.commit(transaction);
            return result;
        } catch (RuntimeException runtimeException) {
            transactionManager.rollback(transaction);
            throw runtimeException;
        }
    }
}
```

> applicationContext.xml

```xml
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="userServiceImpl"/>
    <property name="interceptorNames">
        <list>
            <value>transactionAdvisor</value>
        </list>
    </property>
</bean>
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
    <property name="advice" ref="transactionAdvice"/>
    <property name="pointcut" ref="transactionPointcut"/>
</bean>
<bean id="transactionAdvice" class="springbook.service.TransactionAdvice">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
    <property name="mappedName" value="upgrade*"/>
</bean>
```

![image](https://user-images.githubusercontent.com/56240505/122429076-74d08500-cfcd-11eb-9fc7-1e0d4d2f49d6.png)

* ProxyFactoryBean은 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것이다.
  * 독립적이고, 여러 프록시가 공유할 수 있는 어드바이스와 포인트컷으로 확장 기능을 분리한다.
* 과거에 Proxy를 서비스별로 일일이 만들어주던 번거로움이 크게 해소된다.
  * 어드바이스와 포인트컷 및 그에 따른 조합 어드바이저를 싱글톤 Bean으로 등록해두고 재사용한다.

<br>

## 5. Spring AOP

* 현재 타깃 오브젝트마다 거의 비슷한 내용의 ProxyFactoryBean Bean 설정 정보를 추가해주는 부분이 계속 중복되고 있다.
* ProxyFactoryBean의 장점?
  * 타깃 오브젝트로의 위임 코드와 부가 기능 적용을 위한 코드가 프록시가 구현해야 하는 모든 인터페이스 메소드마다 반복적으로 필요했다.
  * JDK의 다이내믹 프록시는 특정 인터페이스를 구현한 오브젝트에 대해서 프록시 역할을 해주는 클래스를 런타임시 내부적으로 만들어준다.
  * 변하지 않는 타깃으로의 위임과 부가기능 적용 여부 판단 부분은 코드 생성 기법을 이용하는 다이내믹 프록시 기술에 맡긴다.
  * 변하는 부가 기능 코드는 별도로 만들어서 다이내믹 프록시 생성 팩토리에 DI한다.

### 5.1. BeanPostProcessor를 이용한 자동 프록시 생성기

* Bean 후처리기는 Bean 오브젝트가 만들어지고 난 후에, 해당 오브젝트를 다시 가공할 수 있게 해주는 확장 포인트 인터페이스다.
* DefaultAdvisorAutoProxyCreator는 어드바이저를 통한 자동 프록시 생성기다.
* Spring은 후처리기가 Bean으로 등록되어 있으면 Bean 오브젝트가 생성될 때마다 후처리기에 보내서 후처리 작업을 요청한다.
  * 이를 통해 Bean 오브젝트의 일부를 프록시로 포장하고, 프록시를 Bean으로 대신 등록할 수 있다.
* Pointcut 인터페이스는 어드바이스를 적용할 메서드인지 확인하는 기능뿐만 아니라 프록시를 적용할 수 있는 클래스인지 확인하는 기능을 포함한다.
  * DefaultAdvisorAutoProxyCreator는 Advisor 인터페이스를 구현한 Bean들을 전부 찾는다.
  * 등록된 어드바이저들의 포인트컷을 보고 전달받은 Bean이 프록시 적용 대상인지 확인한후, 맞다면 프록시를 생성하고 어드바이저와 연결해준다.
  * 최종적으로 타깃 오브젝트를 감싼 프록시 오브젝트로 바꿔치기 하여 Bean 컨테이너에게 반환한다.

> HelloTest.java

```java
@Test
void classNamePointcutAdvisor() {
    NameMatchMethodPointcut nameMatchMethodPointcut = new NameMatchMethodPointcut() {
        @Override
        public ClassFilter getClassFilter() {
            return new ClassFilter() {
                @Override
                public boolean matches(Class<?> clazz) {
                    return clazz.getSimpleName().startsWith("HelloT");
                }
            };
        }
    };
    nameMatchMethodPointcut.setMappedName("sayH*");

    checkAdviced(new HelloTarget(), nameMatchMethodPointcut, true);
    checkAdviced(new HelloWorld(), nameMatchMethodPointcut, false);
    checkAdviced(new HelloToby(), nameMatchMethodPointcut, true);

}

private void checkAdviced(Hello hello, NameMatchMethodPointcut nameMatchMethodPointcut, boolean isAdviced) {
    ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
    proxyFactoryBean.setTarget(hello);
    proxyFactoryBean.addAdvisor(new DefaultPointcutAdvisor(nameMatchMethodPointcut, new UppercaseAdvice()));
    Hello proxy = (Hello) proxyFactoryBean.getObject();

    if (isAdviced) {
        assertThat(proxy.sayHello("abc")).isEqualTo("HELLOABC");
        assertThat(proxy.sayHi("abc")).isEqualTo("HIABC");
        assertThat(proxy.sayThankYou("abc")).isNotEqualTo("THANK YOUABC");
    } else {
        assertThat(proxy.sayHello("abc")).isNotEqualTo("HELLOABC");
        assertThat(proxy.sayHi("abc")).isNotEqualTo("HIABC");
        assertThat(proxy.sayThankYou("abc")).isNotEqualTo("THANK YOUABC");
    }
}
```

* Pointcut이 클래스 이름에 대해서도 필터링하는 것을 확인할 수 있다.

> NameMatchMethodPointcut.java

```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {

    public void setMappedClassName(String mappedClassName) {
        this.setClassFilter(new SimpleClassFilter(mappedClassName));
    }

    static class SimpleClassFilter implements ClassFilter {

        String mappedName;

        private SimpleClassFilter(String mappedName) {
            this.mappedName = mappedName;
        }

        @Override
        public boolean matches(Class<?> clazz) {
            return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
        }
    }
}
```

> applicationContext.xml

```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
<bean id="userService" class="springbook.service.UserServiceImpl">
    <property name="userDao" ref="userDao"/>
    <property name="userLevelUpgradePolicy" ref="userLevelUpgradePolicy"/>
    <property name="mailSender" ref="mailSender"/>
</bean>
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
    <property name="advice" ref="transactionAdvice"/>
    <property name="pointcut" ref="transactionPointcut"/>
</bean>
<bean id="transactionAdvice" class="springbook.proxy.TransactionAdvice">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
<bean id="transactionPointcut" class="springbook.proxy.NameMatchClassMethodPointcut">
    <property name="mappedName" value="upgrade*"/>
    <property name="mappedClassName" value="*ServiceImpl"/>
</bean>
<bean id="testUserService" class="springbook.service.UserServiceTest$TestUserServiceImpl" parent="userService"/>
```

> UserServiceTest.java

```java
@Test
@DirtiesContext
void upgradeAllOrNothing() throws Exception {
    userDao.deleteAll();
    users.forEach(user -> userDao.addUser(user));

    try {
        testUserService.upgradeLevels();
    } catch (IllegalArgumentException e) {
        System.out.println("hi");
    }

    checkLevel(users.get(1), false);
}

@Test
void advisorProxyCreator() {
    assertThat(testUserService).isInstanceOf(java.lang.reflect.Proxy.class);
}

static class TestUserServiceImpl extends UserServiceImpl {

    private String id = "madnite1";

    @Override
    protected void upgradeLevel(User user) {
        if (user.getId().equals(this.id)) {
            throw new IllegalArgumentException();
        }
        super.upgradeLevel(user);
    }
}
```

* 복잡하던 UserService Bean의 설정을 원상복구시킨다.
* 트랜잭션 롤백 확인 테스트를 위해 생성한 static 테스트 클래스를 Bean으로 등록한다.
  * $를 통해 static 멤버 클래스임을 표기하고, 부모 클래스를 설정함으로써 다른 Bean 설정 내용을 상속받는다.
* 후처리기를 통해 자동으로 ``*ServiceImpl`` 이름의 Bean을 트랜잭션 부가 기능이 담긴 프록시 객체로 바꿔치기하기 때문에, 별도의 ProxyFactoryBean 없이도 프록시가 정상 생성됨을 확인할 수 있다.
* testUserService Bean이 프록시 객체로 바꿔치기 되었다면 타입은 TestUserServiceImpl이 아니라 JDK의 Proxy 클래스가 된다.

### 5.2. 포인트컷 표현식

* 기존 Pointcut 인터페이스를 확장하는 경우 클래스와 메서드에 대한 선정 기준 두 가지를 모두 제공해야 한다.
* AspectJExpressionPointcut 클래스를 사용하면 클래스와 메소드의 선정 알고리즘을 포인트 컷표현식을 이용해 한 번에 지정할 수 있다.

> PointcutTest.java

```java
class PointcutTest {

    @Test
    void methodSignaturePointcut() throws NoSuchMethodException {

        AspectJExpressionPointcut aspectJExpressionPointcut = new AspectJExpressionPointcut();
        aspectJExpressionPointcut.setExpression("execution(public int springbook.learningtest.PointcutTest$Target.minus(int, int) throws java.lang.RuntimeException)");

        assertThat(aspectJExpressionPointcut.getClassFilter().matches(Target.class) &&
                aspectJExpressionPointcut.getMethodMatcher().matches(Target.class.getMethod("minus", int.class, int.class), null))
                .isTrue();

        assertThat(aspectJExpressionPointcut.getClassFilter().matches(Target.class) &&
                aspectJExpressionPointcut.getMethodMatcher().matches(Target.class.getMethod("plus", int.class, int.class), null))
                .isFalse();

        assertThat(aspectJExpressionPointcut.getClassFilter().matches(Bean.class) &&
                aspectJExpressionPointcut.getMethodMatcher().matches(Target.class.getMethod("minus", int.class, int.class), null))
                .isFalse();
    }

    interface TargetInterface {
        void hello();

        void hello(String a);

        int minus(int a, int b);

        int plus(int a, int b);
    }

    static class Target implements TargetInterface {

        public void hello() {
        }

        public void hello(String a) {
        }

        public int minus(int a, int b) throws RuntimeException {
            return 0;
        }

        public int plus(int a, int b) {
            return 0;
        }

        public void method() {
        }
    }

    static class Bean {

        public void method() throws RuntimeException {
        }
    }
}
```

* 표현식에서 옵션 항목을 제외하면 ``execution(int minus(int,int))``과 같은 느슨한 표현식도 가능하다.
* ``execution(* minus(..))`` : 리턴 타입과 파라미터의 개수 및 타입을 신경쓰지 않고 포인트컷을 적용함을 의미한다.
  * ``execution(* *(..))`` : 모든 메서드를 다 허용하는 포인트컷으로 가장 느슨하다.
* 포인트컷 표현식을 이용하는 포인트컷 적용도 가능하다.
  * ``bean(*Service)`` : 아이디가 Service로 끝나는 모든 Bean을 포인트컷 대상으로 선택한다.
  * ``@annotation(org.springframework.transaction.annotation.Transactional)`` : 특정 애너테이션이 붙은 메서드를 포인트컷 대상으로 선정한다.
* 표현식의 ``Target``을 ``TargetInterface``로 변경해도 포인트컷이 잘 적용되며 해당 테스트는 통과한다.
  * 포인트컷 표현식의 클래스 이름에 적용되는 패턴은 클래스 이름 패턴이 아니라 타입 패턴이다.
  * ``execution(* *..*Service.upgrade*(..))`` 표현식은 UserService 인터페이스를 구현한 하위 타입 클래스들도 포인트컷이 적용된다.

> applicationContext.xml

```xml
<aop:config>
    <aop:pointcut id="transactionPointcut" expression="execution(* *..*ServiceImpl.upgrade*(..))"/>
    <aop:advisor advice-ref="transactionAdvice" pointcut-ref="transactionPointcut"/>
</aop:config>
```

* 별도의 AOP 네임 스페이스를 이용해서 포인트컷과 어드바이저를 정의할 수 있다.

### 5.3. AOP

* 현재 UserService는 ``트랜잭션 추상화`` -> ``프록시와 데코레이터 패턴`` -> ``다이내믹 프록시와 프록시 팩토리 빈`` -> ``후처리기를 통한 자동 프록시 생성과 포인트컷`` 순으로 개선되었다.
* 관심사가 같은 코드를 객체지향 설계 원칙에 따라 분리하고, 서로 낮은 결합도를 가진 채로 독립적이고 유연하게 확장할 수 있는 모듈로 만드는 것이 개발의 핵심이다.
  * 런타임에 인터페이스 DI를 통해 느슨한 결합 관계를 가지게 함으로써 대부분 실현 가능했다.
  * 트랜잭션 부가 기능 구현은 다른 모듈의 코드에 부가적으로 부여되는 기능이며, 한데 모을 수 없고 어플리케이션 전반에 여기저기 흩어져 있다.
  * 부가 기능이라 독립적인 방식으로 존재하기 때문에 일반적인 객체지향 방식의 개발로는 모듈화하기 어렵다.
    * 각 기능을 부가할 대상인 각 타깃의 핵심 코드 안에 침투하거나 긴밀하게 연결되어야 한다.
  * AOP를 통해 이런 부가 기능이 Advice의 형태로 독립적인 모듈화가 가능해졌다.
    * 코드는 중복되지 않으며, 변경이 필요하면 한 곳만 수정하면 되고, 핵심 코드와 설정에 영향을 끼치지 않는다.

#### 관점 지향 프로그래밍

![image](https://user-images.githubusercontent.com/56240505/122452559-57f37c00-cfe4-11eb-8446-c199c026b262.png)

* 애플리케이션의 핵심적인 기능에서 부가적인 기능을 분리해서 애스펙트라는 독특한 모듈로 만들어서 설계하고 개발하는 방법이다.
  * 전통적인 객체지향 기술의 설계 방법으로는 독립적인 모듈화가 불가능한 부가 기능을 모듈화하는 방식이다.
* 부가 기능 모듈을 Aspect라고 한다.
  * 핵심기능을 담고 있지는 않지만 애플리케이션을 구성하는 중요 요소이며, 핵심기능에 부가되어 의미를 갖는 특별한 모듈이다.
  * 부가될 기능을 정의한 어드바이스와, 어드바이스를 어디에 적용할 지를 결정하는 포인트컷을 함께 가지고 있다.
* 핵심 기능은 순수하게 그 기능을 담은 코드로만 존재하도록 부가 기능과 분리한다.
* OOP를 보완해주는 역할이다.

#### Proxy 활용

* Spring AOP는 JDK와 IoC Container 외에는 특별한 기술이나 환경을 요구하지 않는다.
* Advice를 확장한 MethodInterceptor 인터페이스는 다이내믹 프록시의 InvocationHandler와 마찬가지로 프록시로부터 메소드 요청정보를 전달받아서 타깃 오브젝트의 메소드를 호출한다.
  * Spring AOP는 Proxy 기반의 AOP다.

#### 바이트코드 활용

* AspectJ는 프록시처럼 간접 방식이 아닌 타깃 오브젝트를 수정하여 부가 기능을 직접 넣어준다.
* 컴파일된 타깃의 클래스 파일 자체를 수정하거나 클래스가 JVM에 로딩되는 시점을 가로채서 바이트코드를 조작한다.
  * 소스 코드는 수정하지 않는다.
* 바이트코드를 조작하면 DI 컨테이너를 통한 자동 프록시 생성 방식을 사용하지 않고도 AOP 적용이 가능하다.
* 프록시 방식보다 훨씬 강력하고 유연한 AOP가 가능하다.
  * 프록시 방식은 클라이언트가 호출할 때 사용하는 메서드로 대상이 제한된다.
  * 바이트코드 조작 방식은 오브젝트 생성, 필드 값의 조회와 조작, 스태틱 초기화 등 다양한 지점에서 부가 기능을 부여할 수 있다.
* 사용하기 위해서는 JVM 실행 옵션을 변경하거나 별도의 바이트코드 컴파일러 혹은 클래스 로더를 사용해야 한다.

#### 용어 정리

* 타깃 : 부가 기능을 부여할 대상으로서 다른 프록시 오브젝트가 될 수도 있다.
* 어드바이스 : 타깃에게 제공할 부가기능을 담은 모듈이다.
* 조인트 포인트 : 어드바이스가 적용될 수 있는 위치다.
  * Spring AOP의 조인트 포인트는 메서드의 실행 단계뿐이다.
* 포인트컷 : 어드바이스를 적용할 조인 포인트를 선별하는 작업 또는 그 기능을 정의한 모듈이다.
  * Spring AOP는 조인트 포인트가 메서드 실행 단계이기에, 포인트컷은 대상 메서드를 선정하는 알고리즘이다.
* 프록시 : 클라이언트와 타깃 사이에 투명하게 존재하면서 부가기능을 제공하는 오브젝트다.
* 어드바이저 : 포인트컷과 어드바이스를 하나씩 갖고 있는 오브젝트다.
* 애스펙트 : AOP의 기본 모듈로서 한 개 또는 그 이상의 포인트컷과 어드바이스의 조합이다.
  * 싱글톤 형태의 오브젝트로 존재한다.

<br>

## 6. Transaction

> TransactionAdvice.java

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition())
```

* DefaultTransactionDefinition이 구현하는 TransactionDefinition 인터페이스는 트랜잭션의 동작 방식에 영향을 주는 네 가지 속성을 정의한다.
* TransactionDefinition을 DI받으면 원하는 속성을 지정할 수 있으나, 해당 트랜잭션 어드바이스를 사용하는 모든 트랜잭션들의 속성이 일괄 변경된다.
  * TransactionInterceptor를 통해 간단히 TransactionAdvice를 구현할 수 있다.

### 6.1. 트랜잭션 전파

* 트랜잭션 경계를 가진 코드에 대해 이미 진행 중인 트랜잭션이 어떻게 영향을 미칠 수 있는가를 정의하는 것이다.
* PROPAGATION_REQUIRED : 진행 중인 트랜잭션이 없으면 새로 시작하고, 이미 시작된 트랜잭션이 있으면 이에 참여한다.
  * DefaultTransactionDefinition의 트랜잭션 전파 속성이다.
* PROPAGATION_REQUIRES_NEW : 앞에서 시작된 트랜잭션이 있든 없든 상관없이 새로운 트랜잭션을 만들어서 독자적으로 동작한다.
* PROPAGATION_NOT_SUPPORTED : 진행 중인 트랜잭션이 있어도 무시하며, 트랜잭션 없이 동작하게 만든다.
* ``getTransaction()``은 트랜잭션 전파 속성과 현재 진행 중인 트랜잭션이 존재하는지 여부를 분석해서 트랜잭션을 가져온다.
  * 그 외에도 다른 전파 옵션들이 존재한다.

### 6.2. 격리 수준

* 여러 개의 트랜잭션들이 동시에 진행되면서 상호간에 어느 정도 영향을 끼칠 수 있는지 정도를 설정하는 옵션이다.
  * DefaultTransactionDefinition은 DataSource에 정의된 ISOLATION_DEFAULT 수준을 따른다.
* 트랜잭션의 제한 시간을 설정할 수 있다.
* 읽기 전용을 설정하면 의도하지 않은 쓰기 작업을 막고, 기술에 따라서 성능 향상이 도모된다.
  * 격리 수준에 따라 조회도 반드시 트랜잭션 안에서 진행될 필요가 있기 때문에 읽기 작업도 트랜잭션을 적용하는 것이 좋다.

### 6.3. TransactionInterceptor

* TransactionInterceptor는 Spring이 제공하며, 트랜잭션 정의를 메소드 이름 패턴을 이용해서 다르게 지정할 수 있는 방법을 제공한다.
* ``rollbackOn()`` 속성을 통해 기본 원칙과 다른 예외 처리를 가능하게 한다.
  * 기본적으로 런타임 예외가 발생하면 트랜잭션은 롤백된다.
  * 반면 체크 예외는 일종의 비즈니스 로직에 따른 의미있는 리턴 방식으로 인식해서 커밋해버린다.
    * Spring은 복구 가능성이 있는 예외 상황은 체크 예외를 사용한다고 가정하기 때문이다.

> applicationContext.xml

```xml
<bean id="transactionAdvice" class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly,timeout_30</prop>
            <prop key="upgrade*">PROPAGATION_REQUIRES_NEW,ISOLATION_SERIALIZABLE</prop>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

* TransactionInterceptor는 메서드 이름 패턴에 부합하는 트랜잭션의 경우 사용할 특정한 전파 및 격리 옵션 등을 TransactionAttributes에 Key-Value로 설정한다.
  * 전파 옵션은 필수고 나머지는 선택 옵션이며, 생략하는 경우 DefaultTransactionDefinition의 속성을 따른다.
  * +나 -로 시작하는건 기본 원칙을 따르지 않는 예외를 설정하는 것이다.
* 패턴이 여러 개가 일치하는 경우 가장 구체적인 패턴을 적용한다.

> applicationContext.xml

```xml
<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="get*" propagation="REQUIRED" read-only="true" timeout="30"/>
        <tx:method name="upgrade*" propagation="REQUIRES_NEW" isolation="SERIALIZABLE"/>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

* Transaction 관련 네임 스페이스를 적용할 수 있다.

> applicationContext.xml

```xml
<aop:config>
    <aop:advisor advice-ref="transactionAdvice" pointcut="bean(*Service)" />
    <aop:advisor advice-ref="batchTxAdvice" pointcut="execution(a.b.*BatchJob.*.(..))" />
</aop:config>
<tx:advice id="transactionAdvice">
  <tx:attributes>
      <tx:method name="get*" propagation="REQUIRED" read-only="true" timeout="30"/>
      <tx:method name="*" propagation="REQUIRED"/>
  </tx:attributes>
</tx:attributes>
<tx:advice id="batchTxAdvice">
    <tx:attributes>...</tx:attributes>
</tx:attributes>
```

* 트랜잭션 포인트컷 표현식은 최소한의 타입 패턴이나 Bean 이름을 사용한다.
  * 메소드나 파라미터, 예외에 대한 패턴을 정의하지 않는게 바람직하다.
* 공통된 메소드 이름 규칙을 통해 최소한의 트랜잭션 어드바이스와 속성을 정의한다.

![image](https://user-images.githubusercontent.com/56240505/122463103-2a143480-cff0-11eb-8bc4-0206fd84aac3.png)

* 프록시 방식의 AOP에서는 프록시를 통한 부가기능의 적용은 클라이언트로부터 호출이 일어날 때만 가능하다.
* 타깃 오브젝트 내로 들어와서 타깃 오브젝트의 다른 메서드를 호출하는 경우에는 프록시를 거치지 않고 직접 타깃의 메소드가 호출된다.
  * 따라서 2번의 경우 ``update()`` 메서드에 해당하는 트랜잭션은 적용되지 않는다.

### 6.4. 트랜잭션 속성 적용

* 일반적으로 특정 계층의 경계를 트랜잭션 경계와 일치시키는 것이 바람직하며, 서비스 계층이 가장 적절하다.
  * 여러 계층에서 트랜잭션 경계 설정 부가 기능을 사용하는 것은 좋지 않다.
  * 서비스 계층이 트랜잭션을 관장하는 만큼, 다른 계층이나 모듈이 DAO에 직접 접근하지 못하도록 한다.
    * DAO에 직접 접근하면 트랜잭션 원자성이 지켜지지 않을 수 있으며 유효성 검사 로직을 피할 위험이 있다.

<br>

## 7. @Transactional

![image](https://user-images.githubusercontent.com/56240505/122466303-1b2f8100-cff4-11eb-8653-bd056842b3e1.png)

* @Transactional 애노테이션은 메소드와 타입(클래스 및 인터페이스) 타깃에 사용이 가능하다.
* Spring은 @Transactional이 붙은 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다.
  * 이 때 사용되는 TransactionAttributeSourcePointcut은 스스로 표현식과 같은 선정기준을 갖고 있진 않다.
  * 대신 @Transactional이 타입 레벨이든 메소드 레벨이든 상관없이 부여된 Bean 오브젝트를 모두 찾아서 포인트컷의 선정 결과로 돌려준다.
* 트랜잭션 속성 정의와 더불어서 포인트컷의 자동 등록에도 사용된다.
  * 트랜잭션 속성을 가져오는 AnnotationTransactionAttributeSource를 사용한다.
* 항상 메서드에 부여된 @Transactional이 가장 우선이기 때문에 @Transactional이 붙은 메서드는 클래스 레벨의 속성을 무시하고 메서드 레벨의 속성을 사용한다.
  * 반면에 @Transactional을 붙이지 않은 메서드는 클래스 레벨에 부여된 공통 @Transactional을 따른다.
  * 타깃 클래스에도 @Transactional이 없으면 클래스의 인터페이스에도 똑같이 @Transactional을 메서드 레벨부터 탐색한다.
  * @Transactional이 없으면 트랜잭션 적용 대상이 아니라고 판단한다.

![image](https://user-images.githubusercontent.com/56240505/122468162-4f0ba600-cff6-11eb-8399-2e2b397f1948.png)

* 전파 옵션을 통해 다양한 크기의 트랜잭션을 만들 수 있다.
* 전파 옵션이 없었더라면 ``add()``는 항상 독자적인 트랜잭션을 만들기 때문에, ``add()``의 코드를 사용자 등록 로직이 포함된 상위 트랜잭션에 항상 복붙해야 한다.
  * 유지보수하기 매우 어렵다.
* AOP를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 속성을 지정하는 방법을 선언적 트랜잭션이라고 한다.
  * EJB도 선언적 트랜잭션을 지원하지만 특정 트랜잭션 기술 및 환경에 종속적이다.

> UserService.java

```java
@Test
void transactionReadonly() {
    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    definition.setReadOnly(true);
    TransactionStatus transaction = transactionManager.getTransaction(definition);
    try {
        userService.deleteAll();
        userService.add(users.get(0));
        userService.add(users.get(1));
        transactionManager.commit(transaction);
    } catch (RuntimeException e) {
        transactionManager.rollback(transaction);
        assertThat(e).isInstanceOf(TransientDataAccessException.class);
    }
}

@Test
void transactionSync() {
    userDao.deleteAll();
    assertThat(userDao.getCount()).isZero();

    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    TransactionStatus transaction = transactionManager.getTransaction(definition);

    userDao.addUser(users.get(0));
    assertThat(userDao.getCount()).isEqualTo(1);

    transactionManager.rollback(transaction);
    assertThat(userDao.getCount()).isZero();
}
```

* 코드에 트랜잭션 관련 API를 직접 사용하는 방식을 프로그램에 의한 트랜잭션 방식이라 한다.
* **선언적인 방식이 사용하기 편하지만, 정밀한 테스트를 위해 PlatformTransactionManager를 직접 사용할 수 있다.**
  * 트랜잭션 결과나 상태를 조작하면서 테스트하는 것도 가능하다.
* 특히 롤백 테스트는 매우 유용하다.
  * 쓰기 관련 메서드 테스트를 하다보면 테스트 전역으로 사용할 샘플용 데이터가 엉망이 되는 경우가 많다.
  * 롤백 테스트는 테스트를 진행하는 동안에 조작한 데이터를 모두 롤백하고 테스트를 시작하기 전 상태로 만들어준다.
    * @JdbcTest, @JpaDataTest 등의 원리다.
  * 각 테스트간의 독립성을 보장해줄 수 있다.
    * 테스트가 진행되는 순서나 앞 테스트 결과 여부가 다른 테스트에 영향을 줘서는 안 된다.
* 롤백 테스트는 여러 개발자가 하나의 공용 테스트용 DB를 사용할 수 있게 해준다.

> UserService.java

```java
@Test
@Transactional(readOnly = true)
void transactionReadonly() {
    assertThatCode(() -> {
        userService.deleteAll();
        userService.add(users.get(0));
        userService.add(users.get(1));
    }).isInstanceOf(TransientDataAccessException.class);
}
```

* 테스트에서도 @Transactional을 사용할 수 있다.
* 테스트용 @Transactional은 테스트 수행후 강제 롤백된다.
  * 테스트 목적으로 만들어진 애너테이션이 아니기에 별도로 롤백 테스트에 대한 설정을 담을 수 없다.
* 롤백을 원하지 않는다면 테스트 결과를 커밋하고 싶다면 ``@Rollback(false)`` 애너테이션을 붙여준다.

> UserService.java

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/test-applicationContext.xml")
@Transactional
@TransactionConfiguration(defaultRollback=false)
public class UserServiceTest {

    @Test
    @Rollback
    public void add() throws SQLException { ... }
}
```

* @TransactionConfiguration을 사용하면 클래스 레벨에서 테스트 메서드의 롤백에 대한 공통 속성을 지정 할 수 있다.
* 그 중 롤백을 적용하고 싶은 일부 메서드만 @Rollback을 붙인다.
* 클래스 레벨에서 @Transactional을 적용했을 때 트랜잭션이 필요없는 일부 테스트 메서드가 존재하는 경우?
  * ``@Transactional(propagation=Propagation.NEVER)``을 명시해주면 트랜잭션이 시작조차 되지 않는다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
