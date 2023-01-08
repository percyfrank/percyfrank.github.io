---
title:  "[Spring Boot 개념과 활용] 23장 : Redis & MongoDB & Neo4j"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-13T08:13:00-05:00
---

## 1. Redis

> Docker

```sh
docker run -p 6379:6379 --name redis_boot -d redis
docker exec -i -t redis_boot redis-cli

hgetall accounts:264b260d-be99-4fc4-bd07-ff1bcb7a3cbc
1) "_class"
2) "com.example.glenn.Account"
3) "id"
4) "264b260d-be99-4fc4-bd07-ff1bcb7a3cbc"
5) "userName"
6) "jipark2"
7) "email"
8) "jipark@github.com"
```

> RedisRunner.java

```java
@Component
public class RedisRunner implements ApplicationRunner {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Autowired
    private AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = stringRedisTemplate.opsForValue();

        values.set("jipark", "glenn");
        values.set("springboot", "2.0");
        values.set("hello", "world");

        Account account = new Account();
        account.setUserName("jipark2");
        account.setEmail("jipark@github.com");

        accountRepository.save(account);
        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getUserName());
        System.out.println(byId.get().getEmail());
    }
}
```

> Account.java

```java
@RedisHash("accounts")
public class Account {

    @Id
    private String id;
    ...
}
```

> AccountRepository.java

```java
public interface AccountRepository extends CrudRepository<Account, String> {}
```

> application.properties

```properties
spring.redis.host=192.168.99.100
spring.redis.port=6379
//Docker 일반 버전이 아닌 ToolBox인 경우 localhost가 잡히지 않음.
//spring.redis로 커스터마이징 가능.
```

* 캐시, 메시지 브로커, 키/밸류 스토어 등으로 사용 가능하다.
* 의존성 :  spring-boot-starter-data-redis.
* Spring Boot에서는 6379 port를 redis 기본 port로 잡아, 일반적인 경우 properties 설정이 필요없다.
* Redis Command.
  * keys \*.
  * get {key}.
  * hgetall {key}.
  * hget {key} {column}.
* Hash(Entity)로 넣은 값은 keys 조회 시 hash get으로 특정 필드를 참조하거나, 모두 조회를 해야한다.

<br>

## 2. MongoDB 및 Neo4j 내용 추가 예정

* Docker를 통해 DB를 띄우고, Spring Boot와 연동하는 방법은 유사하다.
* 나중에 NoSQL DB를 사용할 때 동영상 강의 및 레퍼런스를 참고하여 내용을 추가할 계획.
  * [레퍼런스](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-sql)

<br>

---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
