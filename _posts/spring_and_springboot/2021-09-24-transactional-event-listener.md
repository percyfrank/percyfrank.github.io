---
title: "@TransactionalEventListener 이벤트 발행을 통한 비즈니스 로직 분리"
excerpt: "이벤트 발행을 통해 외부 모듈 간의 결합도를 낮춘다."
categories:
  - Spring & Spring Boot
tags:
  - Spring & Spring Boot
date: 2021-09-24
last_modified_at: 2021-09-24
---

## 1. 트랜잭션 내부의 외부 모듈 연동

> ExampleUserService.java

```java
@Transactional
public Long save(UserCreateRequestDto requestDto) {
    User user = userRepository.save(requestDto.toEntity());
    mailService.sendMail(MailDto.from(user));
    return user.getId();
}
```

서비스를 개발하다 보면 외부 모듈 및 시스템 API를 연동하는 경우가 많아진다. 위 예제는 회원 가입시 가입 축하 메일을 발송하는 코드다.

* 회원 가입 기능과 메일 발송 기능이 하나의 트랜잭션으로 묶여있는 단점이 존재한다.
  * 메일 발송을 위해 사용하는 외부 SMTP 서버가 느려지면 회원 가입 트랜잭션 처리 속도가 느려진다.
  * 메일 서버에 장애가 발생해서 메일 발송이 실패하면, 트랜잭션 자체가 롤백되어 회원 가입에 실패한다.
* 클라이언트의 요청 목적은 **회원 가입**이지만, 요청의 의도와 다른 로직으로 인해 트랜잭션이 느려지거나 실패하면 문제로 인식되어야 한다.
  * 회원 가입 트랜잭션이 종료되고 나서, 가입 축하 메일이 발송되어야 한다.

> UserService.java

```java
@Transactional(readOnly = true)
@Service
public class UserService {

    private final UserRepository userRepository;
    private final GitHubFollowingRequester gitHubFollowingRequester;

    public UserService(
        UserRepository userRepository,
        GitHubFollowingRequester gitHubFollowingRequester
    ) {
        this.userRepository = userRepository;
        this.gitHubFollowingRequester = gitHubFollowingRequester;
    }

    @Transactional
    public FollowResponseDto followUser(FollowRequestDto requestDto) {
        User source = findUserByName(requestDto.getUsername());
        User target = findUserByName(requestDto.getTargetName());
        source.follow(target);
        gitHubFollowingRequester.follow(
            requestDto.getTargetName(),
            requestDto.getAccessToken()
        );
        return generateFollowResponse(target, true);
    }
}
```

이번에 팀 프로젝트를 진행하면서 리팩토링을 하게 된 서비스 예제 코드다. 어플리케이션에서 특정 유저를 팔로우 혹은 언팔로우했을 때, 외부 모듈을 통해 연동된 GitHub 계정을 팔로우 혹은 언팔로우 하는 로직이다. 회원 가입시 가입 축하 메일을 발송하는 예제와 비슷한 문제를 가지고 있다.

<br>

## 2. @TransactionalEventListener

> FollowEvent.java

```java
public class FollowEvent {

    private final String targetName;
    private final String accessToken;

    public FollowEvent(String targetName, String accessToken) {
        this.targetName = targetName;
        this.accessToken = accessToken;
    }

    // getter
}
```

* 굳이 ApplicationEvent를 상속받을 필요가 없다.

> EventListener.java

```java
@RequiredArgsConstructor
@Component
public class EventListener {

    private final GitHubFollowingRequester gitHubFollowingRequester;

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT, fallbackExecution = true)
    public void handleFollowEvent(FollowEvent followEvent) {
        gitHubFollowingRequester.follow(
            followEvent.getTargetName(),
            followEvent.getAccessToken()
        );
    }
}
```

* phase는 발생한 이벤트를 리스너가 바인딩할 단계를 설정하는 값이다.
  * BEFORE_COMMIT, AFTER_COMMIT, AFTER_ROLLBACK, AFTER_COMPLETION 등.
* fallbackExecution 값이 true인 경우, 트랜잭션이 아닌 곳에서 이벤트를 발행하면 예외가 발생한다.

> UserService.java

```java
@Transactional(readOnly = true)
@Service
public class UserService {

    private final UserRepository userRepository;
    private final ApplicationEventPublisher applicationEventPublisher;

    public UserService(UserRepository userRepository,
        ApplicationEventPublisher applicationEventPublisher) {
        this.userRepository = userRepository;
        this.applicationEventPublisher = applicationEventPublisher;
    }

    @Transactional
    public FollowResponseDto followUser(FollowRequestDto requestDto) {
        User source = findUserByName(requestDto.getUsername());
        User target = findUserByName(requestDto.getTargetName());
        source.follow(target);
        FollowEvent followEvent = new FollowEvent(
            requestDto.getTargetName(),
            requestDto.getAccessToken()
        );
        applicationEventPublisher.publishEvent(followEvent);
        return generateFollowResponse(target, true);
    }
}
```

* 트랜잭션 내부에서는 이벤트만 발행하고, 트랜잭션이 커밋되는 시점에 리스너가 해당 이벤트를 처리하도록 @TransactionalEventListener를 적절하게 정의해준다.
* 이벤트 발행을 통해 외부 모듈 연동 로직을 특정 트랜잭션과 분리하여 수행할 수 있으며, 이는 모듈 간의 결합도를 낮추는 역할을 한다.
  * 더이상 UserService가 메일 전송이나 팔로우 수행 등 외부 모듈에 직접적으로 의존하지 않게 된다.

<br>

---

## References

* [이벤트 발행으로 비즈니스 로직 분리하기](https://tecoble.techcourse.co.kr/post/2020-09-30-event-publish/)
