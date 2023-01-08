---
title: "[토비의 스프링] 4장. 예외"
excerpt: "스프링은 DataAccessException이라는 추상화된 런타임 예외 계층을 제공한다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
toc: true
toc_sticky: true
last_modified_at: 2021-05-18
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter4)

## 1. 예외

> UserDao.java

```java
public User findUser(Long id) {
    try {
        //...
    } catch (SQLException ignored) {
        ignored.printStackTrace();
    }
}


public User findUser(Long id) throws Exception {
    //...
}
```

* CheckedException에 대한 catch문에 아무런 작업을 하지 않거나, 단순히 stackTrace 콘솔 출력만 지시해두면 문제 상황에서 원인을 찾기 어렵다.
  * 예외가 발생하지 않은 것 처럼 넘어가버리면, 오류가 발생해도 프로그램이 계속 진행되다가 다른 곳에서 예상하지 못한 문제가 발생할 수 있다.
  * 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자에게 로그 파일 등을 통해 분명하게 통보되어야 한다.
* 예외를 잡아 조치를 취할 방법이 없다면 잡지 않고 메서드 외부로 전파한다.
  * 정확한 예외명 대신 추상적인 상위 Exception을 도배하는 것은 바람직하지 못하다.
  * 적절한 처리를 통해 복구될 수 있는 예외상황도 제대로 다룰 수 있는 기회를 박탈하는 셈이다.

### 1.1. 종류

* Error
  * JVM에서 주로 발생시키는 Error 클래스의 서브 클래스이며, 시스템에 비정상적인 상황이 발생한경우 사용된다.
  * 메모리, 스레드 등의 이슈는 어플리케이션 레벨에서 대응할 수 없으니 catch로 잡지 않는다.
* Exception
  * Exception 클래스와 서브 클래스들로, 개발자가 작성한 어플리케이션 코드에서 예외 상황이 발생할 때 사용된다.
  * CheckedException은 Exception을 상속한 예외이며, UncheckedException은 Exception의 서브 클래스인 RuntimeException을 상속한 예외다.
    * CheckedException은 예외 처리를 강제한다.

### 1.2. 처리 방법

> ExceptionHandler.java

```java
int maxretry = MAX_RETRY;
while (maxretry-- > 0) {
    try {
      ... // 예외가 발생할 가능성이 있는 시도
      return; // 작업 성공
    } catch(SomeException e) {
      // 로그 출력. 정해진 시간만큼 대기
    } finally {
      // 리소스 반납. 정리 작업
    }
}
throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```

* 예외 복구
  * 예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는다.
  * 단순히 예외 메시지를 사용자에게 돌려주는 것이 아니라, 정해진 횟수만큼 재시도를 통해 문제 상황을 복구한다.
    * 예외를 통해 복구 가능성이 있을 때 CheckedException을 사용한다.

> UserDao.java

```java
public User findUser(Long id) throws SQLException {
    try {
        //...
    } catch (SQLException e) {
        //로그 출력
        throw e;
    }
}
```

* 예외 처리 회피
  * 호출한 쪽으로 예외를 전파하거나, catch로 예외를 잡아 로그를 남긴 뒤 다시 예외를 던지는 방식이다.
  * catch 블럭에서 아무 처리를 하지 않고 예외가 발생하지 않은 것처럼 만드는 것과는 다르며, 다른 곳에서 예외를 대신 처리하도록 만드는 것이다.
* JdbcTemplate의 PreparedStatement나 ResultSet 등을 이용하다 발생하는 SQLException은 이를 호출한 템플릿에서 처리하도록 던진다.
  * 그러나 무작정 Dao가 외부로 SQLException을 전파하는 것은 지양해야 한다.
    * 서비스 계층이나 웹 계층에서는 해당 예외를 적절하게 처리할 수 없다.
* 예외를 회피하는 것은 예외 복구처럼 의도가 분명해야 한다.
  * 콜백/템플릿과 같이 긴밀한 관계의 오브젝트가 예외 처리 책임을 분명하게 지게 하거나, 자신을 사용하는 쪽에서 예외를 확실히 핸들링할 수 있다는 확신이 있어야 한다.

> UserDao.java

```java
public User findUser(Long id) throws DuplicationUserIdException, SQLException {
    try {
        //...
    } catch (SQLException e) {
        if (e.getErrorCode() == MySqlErrorNumbers.ER_DUP_ENTRY) {
            throw new DuplicationUserIdException(e);
            //throw new DuplicationUserIdException().initCause(e);
        }
        throw new RuntimeException(e);
    }
}
```

* 예외 전환
  * 예외 회피와 마찬가지로 예외 복구가 불가능한 경우, 예외를 적절한 예외로 전환해 메서드 밖으로 던진다.
  * IO 등 시스템의 로우 레벨 예외가 예외 상황의 의미를 분명하게 표현하지 못할 때, 의미를 분명하게 하는 예외로 번역한다.
    * 로우 레벨 SQLException을 외부로 전파하면, 서비스 계층 등은 원인을 파악하기 어렵다.
    * 정보를 해석해서 기술에 독립적인 적합한 의미의 추상화된 예외를 반환하면, 외부 서비스 계층에서도 쉽게 복구 작업을 시도할 수 있다.
    * 서비스 계층이 SQLException을 분석할 수 있으나, 특정 기술의 정보를 해석하는 코드를 비즈니스 로직 계층에 담는 것은 어색하다.
  * 예외 전환의 경우 원인 예외를 담아 중첩 예외를 발생시키며, ``getCause()``를 통해 처음 예외를 확인할 수 있다.
  * DuplicationUserIdException은 꼭 Dao 뿐만 아니라 다양한 곳에서 사용할 수 있기 때문에 런타임 예외로 지정한다.
    * 다만 UserDao를 사용하는 쪽에서 예외 처리를 할 때 도움을 주기 위해, throws 절에 해당 예외를 던진다고 표기하는 것이 바람직하다.

> EJBExceptionTest.java

```java
try {
    OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
    Order order = orderHome.findByPrimaryKey(Integer id);
} catch (NamingException ne) {
    throw new EJBException(ne);
} catch (SQLException se) {
    throw new EJBException(se);
} catch (RemoteException re) {
    throw new EJBException(re);
}
```

* CheckedException이 비즈니스 로직으로 볼 때 의미없거나 복구 가능한 예외가 아닌 경우, 런타임 예외로 감싸 던진다.
  * 런타임 예외이기 때문에 위 코드의 컴포넌트를 이용하는 다른 클라이언트는 일일이 예외를 잡을 필요가 없다.
    * 복구할 방법이 없기 때문이다.
* 어플리케이션 로직상에서 예외 상황이 발생하는 경우, API가 아닌 어플리케이션 코드에서 의도적으로 던지는 예외다.
  * 이 때는 비즈니스적인 의미가 있는 예외이기 때문에 CheckedException을 사용해 적절한 대응 및 복구를 진행해야 한다.
* 복구 불가능한 예외라면 상위 계층까지 throws로 예외를 불필요하게 전파하지 말고, 런타임 예외로 포장한다.
  * throws를 메서드에 무의미하게 선언하는 것은 가독성을 낮춘다.
  * 대부분의 웹 어플리케이션에 내장된 예외 처리 서비스를 통해 로그 파일을 남기고 사용자에게 통보하는 등으로 처리하는게 바람직하다.

### 1.3. 런타임 예외의 보편화

* 워드와 같은 프로그램(독립형 어플리케이션)은 통제 불가능한 예외라고 할지라도 어플리케이션의 작업이 중단되지 않게 해주고 상황을 복구해야 한다.
  * 파일 열기 기능에서 해당 파일이 없다고 프로그램이 종료되면 안 된다.
* Java 엔터프라이즈 서버 환경은 수 많은 사용자가 동시에 보낸 요청이 각각 독립적인 작업으로 취급된다.
  * 요청 처리 도중 예외가 발생하면 해당 작업만 중지하면 된다.
  * 독립형 어플리케이션과 달리 예외가 발생했을 때, 작업을 일시 중지하고 클라이언트와 커뮤니케이션하며 복구할 수 없기 때문이다.
  * 예외가 안 생기도록 하는 것이 가장 좋으며, 발생하면 빠르게 요청을 취소하고 개발자에게 통보하는 것이 좋다.
    * 따라서 CheckedException의 활용도가 점차 줄어들고 있으며, API들은 CheckedException을 주로 사용했으나 최근에는 런타임 예외를 사용하는 추세다.
    * CheckedException은 무분별한 예외 전파의 원인이다
    * 복구할 수 있는 가능성이 조금이라도 존재하면 CheckedException을 사용했으나, 요즘은 항상 복구할 수 있는 예외가 아니면 런타임 예외를 사용한다.
* 단, 컴파일러가 예외 처리를 강제하지 않기 때문에 예외의 종류와 원인 및 활용 방안을 항상 문서화한다.

> Bank.java

```java
try {
    BigDecimal balance = account.withdraw(amount);
    ...
    // 정상적인 처리 결과를 출력하도록 진행
} catch(InsufficientBalanceException e) { // 체크 예외
    // InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
    BigDecimal availFunds = e.getAvailFunds();
    ...
    // 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
}
```

* 어플리케이션 예외란?
  * 시스템이나 외부가 원인이 아닌, 어플리케이션 자체의 로직에 의해 의도적으로 발생시키고 반드시 catch해 조치를 취하는 예외다.
  * 예외가 없다면 반환값에 따른 if 조건 분기를 통해 로직을 수행해야 하기 때문에, 오히려 이 방식이 가독성이 우수하다.

<br>

## 2. SQLException

SQLException의 대부분은 코드 레벨에서 복구하기 어렵다. 프로그램의 오류나 개발자의 부주의 혹은 통제할 수 없는 외부 상황(DB 서버 다운, 제약 조건 위배, 네트워크 등)에서 기인하기 때문이다.

* 시스템 예외는 어플리케이션 레벨에서 복구하기 어렵다.
  * 잘못된 입력값으로 인해 Dao의 쓰기 작업이 실패한 경우, SQLException을 복구할 바에 런타임 예외로 개발자에게 발생 예외를 분명하게 알린다.
  * 이를 통해, 어플리케이션 단은 입력값 검증을 강화하도록 조치를 빠르게 취할 수 있다.

> JdbcTemplate.java

```java
public int update(final String sql) throws DataAccessException { ... }
```

* Spring JdbcTemplate은 템플릿과 콜백에서 발생하는 SQLException을 런타임 예외로 포장해 던져주기 때문에, Dao 메서드에 불필요한 throws 전파절이 사라졌다.
  * DataAccessException는 예외를 명시하기 위해 전파절에 선언했을 뿐, 런타임 예외다.
  * JdbcTemplate을 사용하는 클라이언트 측에서 필요한 경우 해당 예외를 처리하면 되고, 그 외는 무시해도 좋다.

<br>

## 3. JDBC의 한계

JDBC는 Java를 통해 DB에 접근하는 방법을 추상화된 API로 제공하고, 각 DB 벤더가 JDBC 표준을 따라 만들어진 드라이버를 제공하도록 한다. 따라서 개발자들은 표준화된 JDBC API만 알면 DB 종류에 상관없이 일관되게 코드를 작성할 수 있다. 그러나 다음 두 가지의 한계를 가지고 있다.

* 비표준 SQL
  * DB 벤더사별로 비표준 문법 및 기능을 제공하며, 최적화 기법을 위해 비표준 SQL을 사용하다 보면 특정 DB에 종속된다.
  * 해결법은 항상 표준 SQL만 사용하거나, DB 별로 별도의 Dao를 만들거나, SQL을 외부에 독립시켜 DB에 따라 변경해 사용하는 방법이 있다.
* 호환성 없는 SQLException의 DB 에러 정보
  * SQLException은 발생 원인이 다양한데, 문제는 DB마다 에러의 종류와 원인이 제각각이다.
  * 문제는 이러한 모든 예외들을 SQLException 하나에 담아버리도록 설계되었다.
    * ``getErrorCode()``로 가져올 수 있는 DB 에러 코드는 벤더사별로 다르게 정의되어 있어, DB를 변경할 경우 이를 사용하는 기능이 오작동할 수 있다.
  * 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가졌기 때문에, DB에 독립적인 유연한 코드를 작성하기 어렵다.
    * ``getSQLState()``는 DB 벤더 종류에 상관없이 통일된 규격의 상태 정보를 반환하도록 고안되었지만, JDBC 드라이버는 이를 정확하게 반환하지 않아 위험하다.

<br>

## 4. JdbcTemplate 예외 전환

JdbcTemplate 이전에는 발생된 확인 예외인 SQLException의 상태 코드를 분석해 더 구체적인 예외로 전환하는 방법을 취했다. 그러나 DB 벤더별로 에러 코드가 제각각이기 때문에 DB를 변경하면 동일한 예외로 전환이 불가능하다.

따라서 Spring은 SQLException을 대체하는 DataAccessException 런타임 예외를 정의하며, 원인에 따른 다양한 하위 예외 클래스들을 세분화해두었다. Spring은 DB 별로 에러 코드를 분류해서 Spring이 정의한 예외 클래스와 매핑해놓은 에러 코드 매핑정보 테이블을 만들어두고 이를 이용한다.

> UserDao.java

```java
public void add() throws DuplicateKeyException {...} //고유키 위배로 인해 발생하는 DataAccessException 하위 예외를 명시
```

Spring은 SQLException을 DataAccessException으로 감쌀 때, 드라이버나 메타 정보를 참고하여 DB의 에러 코드를 DataAccessException 계층 구조의 클래스 중 하나로 매핑해준다. 즉, 매핑 정보를 참고해서 적절한 예외 클래스를 선택하기 때문에 DB가 달라져도 같은 종류의 에러라면 동일한 예외를 받을 수 있다.

<br>

## 5. DAO 인터페이스와 DataAccessException 계층 구조

DataAccessException은 JDBC 외에도 JPA 등 다른 표준 데이터 액세스 기술의 종류에 상관없이 일관된 예외가 발생하도록 만들어준다. DataAccessException은 Java의 주요 데이터 액세스 기술에서 발생할 수 있는 대부분의 예외를 추상화하고 있다.

### 5.1. DAO 인터페이스 분리 목적

> UserDao.java

```java
public interface UserDao {

    public void add(User user);
    //...
}
```

* 데이터 액세스 로직을 담은 코드를 분리해놓기 위함이다.
  * 분리된 DAO는 전략 패턴을 적용하기 유용하다.
* DAO를 사용하는 쪽에서 DAO가 내부에서 어떤 데이터 액세스 기술을 사용하는지 신경 쓰지 않아도 된다.
  * 자유롭게 JDBC, JPA 등으로 전환이 가능하다.
* 그러나 아래와 같은 이유로 인해 위 인터페이스를 쉽게 사용하기 어렵다.

> UserDao.java

```java
public interface UserDao {

    public void add(User user) throws SQLException; //JDBC
    public void add(User user) throws PersistentException; //JPA
    public void add(User user) throws HibernateException; //Hibernate
    public void add(User user) throws JdoException; //JDO
    //JDBC 구현 DAO가 예외를 전파하려면 인터페이스도 위와 같이 변경되어야 한다.
    //...
}
```

* 각 데이터 액세스 API마다 독자적인 예외를 호출하는 예외 불일치 문제가 존재한다.
  * JDBC를 사용하는 DAO 구현을 다른 JPA로 변경한다면, SQLException을 발생시키는 인터페이스는 사용할 수 없다.
  * 인터페이스 메서드 선언에 없는 예외(PersistentException)를 구현 클래스 메소드의 throws에 넣을 수 없기 때문이다.
  * 데이터 액세스 구현 기술마다 메서드 선언이 달라져야 한다는 단점이 명확하다.
* UserDao를 기술에 독립적인 인터페이스로 분리하기 위해서는 CheckedException(SQLException)을 메서드 내부에서 런타임 예외로 전환해 처리한다.
  * 이를 통해 메서드 선언부의 예외 전파절을 삭제할 수 있다.
  * 다행히 JPA 등 다른 데이터 액세스 기술들은 런타임 예외를 주로 사용하기 때문에 throws 선언이 필요없다.

### 5.2. 데이터 액세스 예외의 추상화

데이터 액세스 예외가 어플리케이션에서는 복구 불가능이더라도, 항상 무시해서는 안 된다. 중복 키 에러처럼 비즈니스 로직에서 의미있게 처리할 수 있는 예외도 있으며, 어플리케이션에서 사용하지 않더라도 시스템 레벨에서 의미있게 이를 분류할 필요도 있다.

그러나 데이터 액세스 기술이 달라지면 같은 상황에서도 다른 종류의 예외가 던져진다. DAO의 사용 기술에 따라 예외 처리 방법이 달라져야 하기 때문에 DAO 인터페이스를 분리했더라도 클라이언트는 DAO의 기술에 의존적이다.

* Spring의 DataAccessException은 데이터 액세스 기술에 상관없는 공통적인 예외뿐만 아니라, 일부 기술에서만 발생하는 예외 역시 대부분 계층 구조로 분류해놓았다.
  * 데이터 액세스 기술을 부정확하게 사용했을 때 InvalidDataAccessResourceUsageException이 발생한다.
    * 이는 또한 구체적인 기술에 따라 BadSqlGrammarException, HibernateQueryException, TypeMismatchDataAccessException 등으로 세분화된다.
  * 낙관적인 락킹에 대한 예외 역시 각 데이터 액세스 기술마다 다른 종류의 예외가 발생한다.
    * Spring은 이들을 DataAccessException의 서브클래스인 ObjectOptimisticLockingFailureException로 통일된다.
* 기술에 독립적인 추상화된 예외가 있어야 DAO를 데이터 액세스 기술에서 독립시킬 수 있다.

### 5.3. UserDao 인터페이스 분리

> UserDao.java

```java
public interface UserDao {

    public void addUser(User user);

    public User getUser(String id);

    public void deleteAll();

    public int getCount();

    public List<User> getAll();
}
```

* JDBC를 사용하는 UserDaoJdbc 클래스가 UserDao를 구현하도록 한다.

### 5.4. 주의사항

> UserDaoTest.java

```java
@Test
void duplicateKey() {
    userDao.deleteAll();
    userDao.addUser(user1);

    assertThatCode(() -> userDao.addUser(user1))
            .isInstanceOf(DuplicateKeyException.class);
}
```

* DataAccessException가 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만, 근본적인 한계가 존재한다.
  * 중복 키가 발생하는 경우 데이터 액세스 기술마다 예외가 다른데, DuplicateKeyException은 아직 JDBC를 이용하는 경우에만 발생한다.
  * JPA 등은 다른 예외가 발생하며, Spring은 그 상위 예외인 DataIntegrityViolationException으로 변환하게 된다.
    * 따라서 JPA로 전환하게 된다면 위 테스트는 실패한다.
* DuplicateKeyException 또한 DataIntegrityViolationException를 상속받지만, DataIntegrityViolationException은 중복키 발생 외의 다른 제약 조건 위배 상황에서도 발생한다.
  * 따라서 구체적인 예외를 원하는 경우 DataIntegrityViolationException은 DuplicateKeyException에 비해 이용가치가 떨어진다.
* 데이터 액세스 기술에 상관없이 같은 예외 상황에 항상 같은 예외가 호출될 것이라고 기대해서는 안 된다.
  * 다만 기술에 상관없이 동일한 예외를 얻고 싶다면, 발생하는 예외를 DAO 내부에서 공통된 커스텀 예외로 전환한다.

> UserDaoTest.java

```java
@Test
void sqlExceptionTranslate() {
    try {
        userDao.deleteAll();
        userDao.addUser(user1);
        userDao.addUser(user1);
    } catch (DuplicateKeyException ex) {
        SQLException sqlEx = (SQLException) ex.getRootCause();
        SQLExceptionTranslator set = new SQLErrorCodeSQLExceptionTranslator(this.dataSource);
        assertThat(set.translate(null, null, sqlEx)).isInstanceOf(DataAccessException.class);
    }
}
```

* ``getRootCause()``로 중첩된 예외를 가져온다.
* 현재 DataSource에 부합하는 SQLExceptionTranslator 객체를 생성하고, 발생 원인인 SQLException을 번역하면 DataAccessException가 도출된다.
  * 즉, Spring이 API 작동 도중 예외를 자동으로 전환함을 확인할 수 있다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
