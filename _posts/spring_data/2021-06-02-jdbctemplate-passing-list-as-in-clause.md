---
title: "JdbcTemplate 사용시 List를 SQL IN 구문의 인자로 전달하기"
excerpt: "맵핑하고자 하는 값들의 List를 IN 구문의 파라미터로 간편하게 맵핑해보자."
categories:
  - Spring Data
tags:
  - Spring Data
toc: true
toc_sticky: true
last_modified_at: 2021-06-02
---

## 1. JdbcTemplate

> UserDao.java

```java
public void List<User> find(long targetId1, long targetId2) {
    String query = "select * from USER where id in (?, ?)";
    RowMapper<User> rowMapper = (rs, rowNum) -> new User(rs.getInt("id"), rs.getString("name"));
    return jdbcTemplate.query(query, rowMapper, targetId1, targetId2);
}
```

JdbcTemplate으로 쿼리 구문에 특정 값을 맵핑하려는 경우 ``?``를 사용하고, DML 호출 메서드에 파라미터들을 순서에 맞춰 대입해야 한다. 만약 맵핑하고자 하는 파라미터가 n개라면 쿼리 구문에도 ``?``를 n개나 작성해야 하며, 호출 메서드에 n개의 파라미터들을 순서에 맞춰 일일이 대입해야 한다.

상당히 번거로운 작업이다. 맵핑하고자 하는 값들의 List를 IN 구문의 파라미터로 간편하게 전달해보자.

> UserDao.java

```java
public void List<User> find(List<Long> ids) {
    String inSql = String.join(",", Collections.nCopies(ids.size(), "?"));
    String query = String.format("select * from USER where id in (%s)", inSql);
    RowMapper<User> rowMapper = (rs, rowNum) -> new User(rs.getInt("id"), rs.getString("name"));
    return jdbcTemplate.query(query, ids.toArray(), rowMapper);
}
```

* ``Collections.nCopies(ids.size(), "?")``를 통해 ``?``를 전달하고자 하는 파라미터의 개수(리스트 사이즈)만큼 생성한다.
* ``String.join()``를 통해 ``?`` 문자열 사이에 컴마를 삽입한다.
* 오버로딩된 ``query()`` 메서드 중 Object 배열을 받는 메서드를 사용하면, 파라미터들이 순서대로 맵핑된다.

<br>

## 2. NamedParameterJdbcTemplate

> UserDao.java

```java
public void List<User> find(List<Long> ids) {
    SqlParameterSource parameters = new MapSqlParameterSource("ids", ids);
    String query = "select * from USER where id in (:ids)";
    RowMapper<User> rowMapper = (rs, rowNum) -> new User(rs.getInt("id"), rs.getString("name"));
    return namedJdbcTemplate.query(query, parameters, rowMapper);
}
```

* NamedParameterJdbcTemplate의 경우, MapSqlParameterSource를 통해 List 인자들을 IN 구문의 파라미터로 간편히 맵핑할 수 있다.

<br>

---

## References

* [Using a List of Values in a JdbcTemplate IN Clause](https://www.baeldung.com/spring-jdbctemplate-in-list)
