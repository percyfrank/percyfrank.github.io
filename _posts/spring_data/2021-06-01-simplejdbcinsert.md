---
title: "JdbcTemplate Insert 작업시 PK 자동 반환하기"
excerpt: "SimpleJdbcInsert를 사용해보자."
categories:
  - Spring Data
tags:
  - Spring Data
toc: true
toc_sticky: true
last_modified_at: 2021-06-01
---

## 1. KeyHolder

> LineDao.java

```java
public Long insert(Line line) {
    String query = "insert into line (name, color) values (?, ?)";
    KeyHolder keyHolder = new GeneratedKeyHolder();
    PreparedStatementCreator preparedStatementCreator = (connection) -> {
        PreparedStatement prepareStatement = connection.prepareStatement(query, new String[]{"id"});
        prepareStatement.setString(1, line.getName());
        prepareStatement.setString(2, line.getColor());
        return prepareStatement;
    };
    jdbcTemplate.update(preparedStatementCreator, keyHolder);
    return keyHolder.getKey().longValue();
}
```

오버로딩된 ``update()`` 메서드들 중 PreparedStatementCreator 및 KeyHolder 인자를 받는 메서드는 자동으로 생성되는 PK 반환 기능을 제공한다.

* Connection을 통해 PrepareStatement를 생성하는 작업은 일반적인 Jdbc API 사용법과 같으며, 두 번째 인자로 PK 칼럼명을 명시해준다.
* 쓰기 작업이 성공하면 KeyHolder는 생성된 PK를 반환한다.
* 해당 방식은 Oracle에서는 동작하지만, 다른 플랫폼에서는 경우에 따라 동작하지 않을 수도 있다.

<br>

## 2. SimpleJdbcInsert

> SectionDao.java

```java
private final SimpleJdbcInsert simpleJdbcInsert;

public SectionDao(JdbcTemplate jdbcTemplate, DataSource dataSource) {
    this.jdbcTemplate = jdbcTemplate;
    this.simpleJdbcInsert = new SimpleJdbcInsert(dataSource)
            .withTableName("SECTION")
            .usingGeneratedKeyColumns("id");
}

public Long insert(Line line, Section section) {
    Map<String, Object> params = new HashMap();
    params.put("line_id", line.getId());
    params.put("up_station_id", section.getUpStation().getId());
    params.put("down_station_id", section.getDownStation().getId());
    params.put("distance", section.getDistance());
    return simpleJdbcInsert.executeAndReturnKey(params).longValue();
}
```

SimpleJdbcInsert는 삽입 작업을 간편하게 수행하도록 지원해주는 클래스다. DataSource 혹은 JdbcTemplate으로 SimpleJdbcInsert 객체를 생성할 수 있으며, 이 때 테이블 이름과 자동으로 생성되는 PK 칼럼명을 명시한다. auto_increment 옵션으로 생성된 값이 해당 칼럼명에 자동으로 입력된다.

* 별도의 쿼리문 없이 파라미터들을 Map에 담아 넣어주면 Key값에 해당하는 칼럼명에 Value값을 삽입한다.
* ``executeAndReturnKey()``는 작업 수행과 동시에 자동 생성된 PK(auto_increment)를 반환한다.
* 자동으로 생성(증가)되는 칼럼이 여러 개이거나 숫자 형태가 아닌 경우, ``executeAndReturnKeyHolder()``를 호출하여 KeyHolder를 반환받아 값을 원하는 형태로 적절히 변환한다.

모든 DB가 특정 Java 클래스를 반환할 것이라고 의존해서는 안 되며, 해당 메서드는 Number 클래스를 반환한다. 이를 원하는 타입으로 적절히 변환해서 사용한다.

### 2.1. SqlParameterSource

> ActorDao.java

```java
public void add(Actor actor) {
    SqlParameterSource parameters = new BeanPropertySqlParameterSource(actor);
    Number newId = insertActor.executeAndReturnKey(parameters);
    actor.setId(newId.longValue());
}

public void add(Actor actor) {
    SqlParameterSource parameters = new MapSqlParameterSource()
            .addValue("first_name", actor.getFirstName())
            .addValue("last_name", actor.getLastName());
    Number newId = insertActor.executeAndReturnKey(parameters);
    actor.setId(newId.longValue());
}
```

SqlParameterSource 인터페이스를 통해 삽입할 파라미터 값을 쉽고 간결하게 추출하여 전달할 수 있다. 혹은 메서드 체이닝을 지원하는 MapSqlParameterSource를 사용할 수 있다. BeanPropertySqlParameterSource는 인자로 주어진 객체가 Java Bean 규약을 갖춘 클래스일 때 사용할 수 있다.

* 칼럼명(혹은 쿼리 문자열에 명시된 파라미터명)에 상응되는 getter 메서드를 호출하여 파라미터 값을 추출한다.
  * 위 경우는 별도의 쿼리문이 없어 해당 테이블의 모든 칼럼명에 해당하는 파라미터 값을 추출하려고 시도한다.
* snake_case로 표기된 DB 칼럼명과 camelCase로 표기된 클래스의 필드명(getter 메서드명) 간을 자동으로 변환하여 맵핑한다.

<br>

---

## References

* [Data access with JDBC](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc)
