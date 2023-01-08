---
title: "[토비의 스프링] 3장. 템플릿"
excerpt: "객체지향 설계가 우수하더라도, 항상 네거티브 테스트부터 작성하는 습관을 가지자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
  - 토비의 스프링
toc: true
toc_sticky: true
last_modified_at: 2021-05-17
---

> [실습 Repository](https://github.com/xlffm3/tobi-vol1/tree/chapter3)

## 1. 기존 UserDao의 문제점

DB 커넥션 등 한정된 개수의 자원은 재사용 가능한 풀로 관리한다. 요청이 많은 서버 환경에서는 매번 새로운 리소스를 생성하지 않고, 풀에 미리 만들어둔 리소스를 돌려가며 사용한다. 그러나 할당한 자원이 제때 반환되지 않으면 리소스가 고갈되어 문제가 발생한다.

> UserDao.java

```java
public void deleteAll() throws SQLException {
    Connection connection = dataSource.getConnection();
    PreparedStatement preparedStatement = connection.prepareStatement("delete from users");
    preparedStatement.executeUpdate();
    preparedStatement.close();
    connection.close();
}
```

* PreparedStatement를 반환하는 도중 예외가 발생하면 Connection 자원이 회수되지 못한다.
* 복잡하게 중첩된 try-catch-finally 문법 혹은 try-with-resources로 자원을 회수해야 한다.
* 예외상황에서 리소스를 반납하는지 체크하는 테스트를 일일이 작성하기 어렵기 때문에, 디자인 패턴을 적용해 리팩토링한다.
  * 복잡하게 중첩된 try-catch-finally를 작성하는 과정에서 자원 반납에 실수가 발생하기 쉽다.
  * try-catch-finally에서 Connection을 맺거나 반환하는 중복 부분을 자주 확장 및 변화하는 코드로부터 분리한다.

### 1.1. 템플릿 메서드 패턴

> UserDao.java

```java
public void deleteAll() throws SQLException {
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    try {
        connection = dataSource.getConnection();
        preparedStatement = makeStatement(connection);
        preparedStatement.executeUpdate();
    } catch (SQLException sqlException) {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
            }
        }
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e2) {
            }
        }
    }
}

public abstract PreparedStatement makeStatement(Connection connection);
```

* UserDao를 추상 클래스로 수정하고, 자주 변화하는 부분(PreparedStatement를 생성하는 부분)을 추상 메서드로 분리한다.
* UserDaoDeleteAll, UserDaoGet, UserDaoAdd 등 하위 구체 클래스가 상속받도록 하여, 자주 변화하는 부분을 자유롭게 확장한다.
* 단점은 다음과 같다.
  * UserDao에서 사용하는 JDBC 메서드가 n개이면, n개의 서브 클래스를 만들어야 한다.
  * 확장 구조 및 서브 클래스의 관계가 클래스 레벨에서 컴파일 시점에 결정된다.
    * 관계에 대한 유연성이 떨어진다.

### 1.2. 전략 패턴

> DeleteStatement.java

```java
public class DeleteAllStatement implements StatementStrategy {

    @Override
    public PreparedStatement makeStatement(Connection connection) throws SQLException {
        return connection.prepareStatement("delete from users");
    }
}
```

> UserDao.java

```java
try {
    connection = dataSource.getConnection();
    preparedStatement = new DeleteAllStatement().makeStatement(connection);
    preparedStatement.executeUpdate();
} catch (SQLException sqlException) {
  //..
}
```

* 변하지 않는 부분(컨텍스트)와 변하는 부분(전략)을 별도의 객체로 두고, 이 두 객체들은 클래스 레벨에서는 인터페이스를 통해 의존하도록 한다.
* 다만 구체 클래를 사용하기 때문에 OCP에 부합하지 않는다.
  * 인터페이스 의존을 통해 외부에서 전략을 런타임 시점에 갈아 끼울수 있도록 클라이언트와 컨텍스트를 분리해야 한다.
  * 클라이언트란 앞단에서 컨텍스트가 어떤 전략을 사용하게 할 것인지를 정한다.

> UserDao.java

```java
public void deleteAll() throws SQLException {
    jdbcContextWithStatementStrategy(new DeleteAllStatement());
}

public void jdbcContextWithStatementStrategy(StatementStrategy statementStrategy) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    try {
        connection = dataSource.getConnection();
        preparedStatement = statementStrategy.makeStatement(connection);
        preparedStatement.executeUpdate();
    } catch (SQLException sqlException) {
      //..
    }
}
```

* 완벽하게 클라이언트와 컨텍스트 클래스를 분리하지 않았지만, 엄밀한 전략 패턴의 모습이다.
  * 전략 패턴 또한 DI를 활용해야 OCP의 이점을 완벽하게 누릴 수 있다.
  * 현재는 Dao의 메서드들이 클라이언트이고, ``jdbcContextWithStatementStrategy()`` 메서드에 전략을 제공한다.

### 1.3. 전략 패턴 리팩토링

매번 전략과 관련된 구현 클래스를 생성하면 관리해야할 파일의 개수가 많아진다. 또한 StatementStrategy에 Connection과 더불어 부수적으로 전달해야할 데이터가 존재한다면, 별도의 생성자나 인스턴스 변수를 전략 구현 클래스에 구현해야 한다.

> UserDao.java

```java
public void addUser(User user) {
    jdbcContextWithStatementStrategy(new StatementStrategy() {
        @Override
        public PreparedStatement makeStatement(Connection connection) throws SQLException {
            PreparedStatement preparedStatement = connection.prepareStatement("INSERT INTO USERS (ID, NAME, PASSWORD) VALUES (?, ?, ?)");
            preparedStatement.setString(1, user.getId());
            preparedStatement.setString(2, user.getName());
            preparedStatement.setString(3, user.getPassword());
            return preparedStatement;
        }
    });
}
```

* 로컬 내부 클래스 혹은 익명 클래스는 해당 클래스가 선언된 메서드의 지역 변수를 접근할 수 있어, 인터페이스에 명시된 파라미터 외의 데이터를 참조하여 로직을 수행할 수 있다.

<br>

## 2. 컨텍스트와 DI

> JdbcContext.java

```java
public class JdbcContext {

    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void workWithStatementStrategy(StatementStrategy statementStrategy) {
        //Connection 맺고 로직 수행 및 반환하는 try-catch-finally 부분
    }

    public void executeQuery(String query) {
        workWithStatementStrategy(new StatementStrategy() {
            @Override
            public PreparedStatement makeStatement(Connection connection) throws SQLException {
                return connection.prepareStatement(query);
            }
        });)
    }
}
```

> UserDao.java

```java
public class UserDao {

    private JdbcContext jdbcContext;

    public void addUser(User user) {
        jdbcContext.workWithStatementStrategy(new StatementStrategy() {
            //...
        });
    }
}
```

* 다양한 Dao에서 JdbcContext 관련 메서드를 사용하도록 별도의 클래스로 추출한 다음, Dao가 DataSource가 아닌 JdbcContext에 의존하도록 변경한다.
* 현재는 인터페이스를 통해 런타임시 다양한 의존 오브젝트를 동적으로 선택할 수 없으며, 클래스 레벨에서 구체적인 의존 관계가 형성되어 있다.
  * 엄밀한 의미의 DI는 인터페이스를 통해 클래스 레벨에서 의존 관계가 고정되지 않게 하며, 런타임 시에 의존할 오브젝트를 다이나믹하게 주입하는 것이 맞다.
  * 그러나 넓게 보면 DI는 객체의 생성과 관계 설정에 대한 제어 권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC 개념을 포괄한다.
    * 비록 구체 클래스지만, Spring을 통해 UserDao에게 JdbcContext를 외부에서 주입하기 때문에 DI라고 볼 수 있다.
* JdbcContext는 그 자체로 독립적인 컨텍스트를 제공하는 서비스 객체로서, 내부 구현이 바뀔 가능성이 없기 때문에 구체 클래스더라도 의미가 있다.
  * 상태 정보가 없기 때문에 Spring 컨테이너에 싱글톤으로 등록하여 여러 오브젝트가 사용하는 것이 이상적이다.
  * 또한 Spring에서 Bean으로 등록된 DataSource에 의존하기 때문에, 이를 주입받기 위해서는 JdbcContext 또한 IoC 대상(Bean)으로 등록되어야 한다.

> UserDao.java

```java
public class UserDao {

    private JdbcContext jdbcContext;

    public void setDataSource(DataSource dataSource) {
        this.jdbcContext = new JdbcContext();
        this.jdbcContext.setDataSource(dataSource);
    }
}
```

* JdbcContext를 Bean으로 등록하기 싫다면, UserDao가 직접 JdbcContext에 대한 DI를 수행할 수 있다.
* UserDao가 Bean 레벨에서 의존하는 DataSource를 JdbcContext 객체를 생성할 때 사용하고 버린다.
  * UserDao가 임시로 DI 컨테이너처럼 동작한다.
* 굳이 인터페이스를 두지 않아도 될 만큼 긴밀한 관계를 갖는 Dao와 JdbcContext를 Bean으로 분리하지 않고 DI를 적용하는 방식이다.
  * 그러나 JdbcContext를 싱글톤으로 만들 수 없으며, DI 작업을 위한 부수 코드가 필요하다.

<br>

## 3. 템플릿과 콜백

전략 패턴의 컨텍스트를 템플릿이라 부르고, 익명 클래스와 같은 전략 객체를 콜백이라고 부른다. 템플릿은 고정된 작업 흐름을 가진 재사용되는 코드를 의미한다. 콜백은 템플릿 내에서 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 의미한다. 값을 참조하기 보다는, 특정 로직을 담은 메소드를 실행하기 위해 사용된다. 콜백은 대게 함수형(단일 메소드) 인터페이스를 사용한다.

* 템플릿/콜백 방식에서는 매번 메소드 단위로 사용할 오브젝트를 새롭게 전달받는다는 것이 특징이다.
* 콜백 오브젝트는 내부 클래스로서 자신을 생성한 클라이언트 메소드 내의 로컬 변수 등 정보를 직접 참조할 수 있다.
* 클라이언트와 콜백이 강하게 결합된다는 면에서도 일반적인 DI와 다른 수동 DI다.
* 템플릿/콜백 패턴이란 DI와 객체지향 설계를 적극 응용한 결과다.
* 지네릭스를 응용한 템블릿/콜백 패턴은 Spring 곳곳에서 사용된다.

<br>

## 4. JdbcTemplate

Spring은 JDBC를 이용하는 Dao에서 사용할 수 있는 다양한 템플릿과 콜백을 제공한다. 또한 자주 사용되는 패턴을 가진 콜백은 다시 템플릿에 결합시켜, 간단한 메서드 호출만으로도 사용이 가능하도록 제작되어 있다.

JdbcTemplate은 DataSource를 사용할 때 Connection을 맺거나 반환 등 반복되는 try-catch-finally 코드들을 템플릿으로 분리하고, 꼭 필요한 인자 및 콜백만을 파라미터로 받아 DB 작업을 수행한다. 이를 통해 Dao 코드를 간결하게 만들어준다.

> UserDao.java

```java
public User getUser(String id) throws SQLException {
    return this.jdbcTemplate.queryForObject("select * from users where id = ?", new Object[]{id}, new RowMapper<User>() {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getString("id"));
            user.setName(rs.getString("name"));
            user.setPassword(rs.getString("password"));
            return user;
        }
    });
}
```

* RowMapper가 호출되는 시점에 이미 커서가 1칸 이동했기 때문에 ``next()``를 호출할 필요가 없다.
  * ``queryForObject()`` 메서드는 조회 결과가 1개가 아닌 경우 자동으로 EmptyResulDataAccessException을 호출한다.

### 4.1. 테스트 보완

**항상 네거티브 테스트부터 작성하는 습관을 들이자.** 조회 결과가 없을 때 null, 0, 예외 호출 등 반환 형태가 다양하기 때문이다. 또한 유지보수를 거치면서 조회 결과가 없을 때, 빈 컬렉션을 반환하던 메서드가 고의로 예외를 던지도록 변경될 수 있다. 이렇듯 예외에 대한 기대 동작방식을 미리 테스트해두면, 추후 Dao 코드의 내부 구현이 변경되었을 때 메서드들이 동일한 기능을 유지하는지 쉽게 파악할 수 있다.

### 4.2. UserDao 완성

> UserDao.java

```java
public void addUser(User user) {
    this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)", user.getId(), user.getName(), user.getPassword());
}

public User getUser(String id) throws SQLException {
    return this.jdbcTemplate.queryForObject("select * from users where id = ?", new Object[]{id}, rowMapper);
}

public void deleteAll() {
    jdbcTemplate.update("delete from users");
}

public int getCount() throws SQLException {
    return jdbcTemplate.queryForObject("select count(*) from users", Integer.class);
}

public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(dataSource);
}

public List<User> getAll() {
    return jdbcTemplate.query("select * from users order by id", rowMapper);
}
```

* Connection을 맺거나 반납 및 JDBC API 사용 방식 등에 대한 책임과 관심은 모두 JdbcTemplate에 위임된 상태다.
  * JdbcTemplate이 변경되어도 UserDao 코드에 큰 영향이 없는 등 결합도가 낮다.
* 반면, 사용할 테이블과 필드 정보가 바뀌면 UserDao의 거의 모든 코드가 변경되기 때문에 응집도는 높다.
* UserDao가 구체 클래스에 의존하지만, 사실상 JdbcTemplate은 Spring에서 JDBC를 통해 Dao를 만드는 데 사용되는 표준 기술이다.
  * 거의 변함이 없어 구체 클래스에 의존해도 무방하지만, 결합도를 더 낮추고 싶다면 상위 인터페이스를 DI 받는다.
* Dao 메서드에서 사용하는 SQL 코드를 UserDao 코드가 아닌 외부 리소스에 담아 이를 읽어와 사용하는 방법도 고려할 수 있다.
  * 이렇게 해두면 DB 테이블 및 필드 이름이 변경되거나, SQL 쿼리를 최적화해야 할 때도 UserDao 코드에는 수정이 필요없어진다.

<br>

---

## References

* 토비의 스프링 3.1(이일민 저)
