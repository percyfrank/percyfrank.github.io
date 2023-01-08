---
title: "Spring Security의 PasswordEncorder"
excerpt: "Password 문자열을 간단하게 인코딩해보자."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-20T10:55:00-05:00
---

## 1. PasswordEncorder

Spring Security에서 제공하는 인터페이스로, 인코딩 뿐만 아니라 ``boolean matches(CharSequence var1, String var2)`` 메서드를 통해 rawPassword와 encodedPassword를 비교할 수 있다.

<br>

## 2. 예제

> UserService.java

```java
@Transactional
@Service
public class UserService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private UserRepository userRepository;

    @Transactional(readOnly = true)
    public User authenticate(String email, String password) {
        User user = userRepository.findByEmail(email)
                .orElseThrow(EmailNotFoundException::new);
        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new PasswordInvalidException();
        }
        return user;
    }
}
```

간단하게 passwordEncoder를 Bean으로 등록하여 Service Logic에서 사용할 수 있다.

> SecurityConfig.java

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

실제 PasswordEncoder 구현 객체는 다양하니, 원하는 타입의 인코더를 이용하면 된다.

<br>

---

## References

* [ahastudio GitHub](https://github.com/ahastudio/fastcampus-eatgo)
