---
title: "Spring Data JDBC의 batchUpdate"
excerpt: "데이터를 일괄적으로 모아서 처리하는 배치 메서드에 대해 알아보자."
categories:
  - Spring Data
tags:
  - Spring Data
date: 2021-06-08
last_modified_at: 2021-06-08
---

> [실습 Repository](https://github.com/xlffm3/spring-learning-test/tree/jdbc-batch)

## 1. batchUpdate

> BatchDao.java

```java
@Repository
public class BatchDao {

    private final JdbcTemplate jdbcTemplate;

    public BatchDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public int insert(List<Car> cars) {
        String sql = "insert into car(name, color) values(?, ?)";
        int size = cars.size();
        int resultSize = 0;
        for (int i = 0; i < size; i++) {
            Car car = cars.get(i);
            resultSize += jdbcTemplate.update(sql, car.getName(), car.getColor());
        }
        return resultSize;
    }

    public int[] batchInsert(List<Car> cars) {
        String sql = "insert into car(name, color) values(?,?)";
        return jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Car car = cars.get(i);
                ps.setString(1, car.getName());
                ps.setString(2, car.getColor());
            }

            @Override
            public int getBatchSize() {
                return cars.size();
            }
        });
    }
}
```

배치란 데이터를 실시간으로 처리하는게 아니라, 일괄적으로 모아서 처리하는 작업을 의미한다. 동일한 PreparedStatement에 대한 복수의 호출을 batch할 수 있다. 예제는 insert지만 update 등의 다른 명령어도 가능하다.

* ``BatchPreparedStatementSetter``를 구현하여 ``batchUpdate()`` 메서드의 인자에 넣어주면 된다.
* ``setValues()`` 메서드는 ``getBatchSize()`` 메서드가 명시한 횟수만큼 호출된다.
* ``getBatchSize()`` 메서드는 현재 batch의 크기를 제공한다.

반환값인 int 배열은 각각의 배치 엔트리로 인해 영향받은 행의 개수를 담고 있다.

> BatchDaoTest.java

```java
@JdbcTest
class BatchDaoTest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    private BatchDao batchDao;
    private List<Car> cars;

    @BeforeEach
    void setUp() {
        batchDao = new BatchDao(jdbcTemplate);
        jdbcTemplate.execute("create table if not exists car ( id bigint auto_increment not null, name varchar ( 255 ) not null, color varchar(255) not null, primary key ( id ))");
        cars = new ArrayList<>();
        for (int i = 1; i <= 10000; i++) {
            cars.add(new Car((long) i, "abcav", "adfkadf"));
        }
    }

    @DisplayName("Batch를 사용하지 않고 데이터를 삽입한다.")
    @Test
    void insert() {
        int returnValue = batchDao.insert(cars);
        List<Car> cars = batchDao.findAll();

        assertThat(returnValue).isEqualTo(10000);
        assertThat(cars).hasSize(10000);
    }

    @DisplayName("Batch를 사용하여 데이터를 삽입한다.")
    @Test
    void batchInsert() {
        int returnValue = batchDao.batchInsert(cars)
                .length;
        List<Car> cars = batchDao.findAll();

        assertThat(returnValue).isEqualTo(10000);
        assertThat(cars).hasSize(10000);
    }
}
```

![image](https://user-images.githubusercontent.com/56240505/121642199-2736b280-cacb-11eb-9372-862e2130e651.png)

* 테스트해본 결과, batchInsert의 성능이 일반적인 insert에 비해 더 우수하다.

<br>

## 2. 변형

> BatchDao.java

```java
public int[] batchInsertVariation(List<Car> cars) {
    List<Object[]> batch = new ArrayList<>();
    for (Car car : cars) {
        Object[] values = new Object[]{car.getName(), car.getColor()};
        batch.add(values);
    }
    String sql = "insert into car(name, color) values(?,?)";
    return jdbcTemplate.batchUpdate(sql, batch);
}
```

* 배치 인터페이스 구현이 너무 장황하다면 오버라이딩된 메서드를 이용할 수 있다.

> BatchDao.java

```java
public int[] batchInsertWithNamedJdbc(List<Car> cars) {
    String sql = "insert into car(name, color) values(:name, :color)";
    return namedParameterJdbcTemplate.batchUpdate(sql, SqlParameterSourceUtils.createBatch(cars));
}
```

* namedParameterJdbcTemplate을 사용한다면 별도의 배치 인터페이스 구현없이, ``SqlParameterSourceUtils``를 통해 인자를 간편하게 SqlParameterSource 배열을 만들어낸다.

<br>

## 3. Multiple Batch

> BatchDao.java

```java
public int[][] batchInsertMultiple(List<Car> cars) {
    String sql = "insert into car(name, color) values(?,?)";
    return jdbcTemplate.batchUpdate(sql, cars, 100, (ps, argument) -> {
        ps.setString(1, argument.getName());
        ps.setString(2, argument.getColor());
    });
}
```

Batch가 너무 큰 경우, 작은 Batch들로 나눌 수 있다. ``batchUpdate()``를 여러 번 호출할 수도 있으나, 오버라이딩 된 메서드를 통해 간편하게 구현한다.

* 중간의 정수 인자는 ``batchUpate()``가 사용하는 배치 크기다.
  * 각각의 배치를 생성하기 위한 ``batchUpdate()`` 횟수다.
* Car 리스트의 크기가 10000이면, 100 * 100의 배열이 반환된다.

<br>

---

## References

* [JDBC Batch Operations](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-advanced-jdbc)
