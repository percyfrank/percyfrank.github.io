---
title:  "[Spring Boot 개념과 활용] 24장 : Spring Security"
excerpt: "Inflearn 백기선님의 강의 및 Spring 공식 문서를 참고하여 정리한 필기입니다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
toc: true
toc_sticky: true
last_modified_at: 2020-07-13T08:14:00-05:00
---

## 1. Spring Security

* 웹 시큐리티.
* 메소드 시큐리티.
* 다양한 인증 방법을 지원한다.
  * LDAP, 폼 인증, Basic 인증, OAuth 등.

<br>

## 2. Spring Boot Security 자동 설정

* SecurityAutoConfiguration 및 UserDetailsServiceAutoConfiguration.
  * 특정 Bean이 없을 때 Default 전략(인메모리 User 객체 생성 및 모든 Http 요청에 대한 인증 요구)을 택한다.
  * 커스텀하게 변경하려면 @ConditionalOnMissing에 해당하는 클래스를 구현한다.
    * 레퍼런스 참조.
* spring-boot-starter-security.
  * 스프링 시큐리티 5.* 의존성을 추가한다.
* 모든 요청에 인증이 필요해진다.
* Basic 인증의 형태는 Accept Header에 따라 달라진다.
* 기본 사용자를 생성한다.
  * Username : user
  * Password : 애플리케이션을 실행할 때 마다 랜덤 값이 생성되며, 콘솔에 출력된다.
  * 다음 프로퍼티로 커스텀하게 수정할 수 있다.
    * spring.security.user.name
    * spring.security.user.password
* 인증 관련 각종 이벤트가 발생한다.
  * DefaultAuthenticationEventPublisher Bean을 등록한다.
  * 다양한 인증 에러 핸들러 등록이 가능하다.

<br>

## 2. Spring Boot Security 테스트

* 의존성 : spring-security-test.
* @WithMockUser라는 어노테이션을 통해 가짜 User 객체를 생성해서 테스트를 진행할 수 있다.
  * 이를 통해 status가 authorized 되었는지를 테스트할 수 있다.
  * Spring Security를 적용하게 되면 일반적인 테스트에서 인증 요청으로 인해 테스트가 깨지게 된다.

<br>

## 3. 커스터마이징

> SecurityConfig.java

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/hello").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

* 특정 주소에서는 인증이 없도록 커스텀하게 규정하며, 그 외의 경우 formLogin과 httpBasic을 이용한 인증을 사용한다는 의미이다.
* Spring Security 기능에서 사용하는 패스워드는 인코딩이 되어 있어야하며, Bean으로 등록한다.
  * 인코딩 방식에 대해서는 레퍼런스를 참고하자.

> AccountService.java

```java
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public Account createAccount(String userName, String password) {
        Account account = new Account();
        account.setUserName(userName);
        account.setPassword(passwordEncoder.encode(password));
        return accountRepository.save(account);
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Account> byUserName = accountRepository.findByUserName(username);
        Account account = byUserName.orElseThrow(() -> new UsernameNotFoundException(username));
        return new User(account.getUserName(), account.getPassword(), authorities());
    }

    private Collection<? extends GrantedAuthority> authorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
}
```

* Spring Security에서 제공하는 인터페이스들이 Bean으로 등록되어 있어서 사용이 가능하다.

<br>


---

## References

* 스프링 부트 개념과 활용(백기선, Inflearn)
