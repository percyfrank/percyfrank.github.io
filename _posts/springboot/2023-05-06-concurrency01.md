---
title:  "동시성 문제 해결 - DB Lock 적용" 
excerpt: "입찰에 낙관적/비관적 Lock 적용"

categories:
  - Springboot
tags:
  - Springboot

date: 2023-05-06
last_modified_at: 2023-05-06

---


### 0. Intro
---

현재 진행 중인 프로젝트는 판매 혹은 구매 입찰을 등록하면, 즉시 구매, 즉시 판매를 통해 낙찰되는 방식으로 진행된다.

바로 이 낙찰되는 순간에 동시성 문제가 발생할 수 있다고 생각했다.

낙찰은 1명 만이 가능한데, 여러 명이 동시에 요쳥 했을 때 모두가 낙찰되면 안 되기 때문이다.

**그래서 동시에 요청이 온다면, 1명은 낙찰이, 나머지는 이미 "낙찰된 거래입니다." 라는 메시지를 전달하게끔 진행하고자 한다.**

동시성을 해결하기 위해 DB의 Lock을 거는 방식으로 진행할 수 있다.

우선 DB의 Lock을 적용하기 앞서, 트랜잭션의 격리 수준을 잠시 파악하고 가자.

애플리케이션 대부분은 동시성 처리가 중요하다. 

그래서 보통 4가지 트랜잭션 격리 수준 중에 `READ COMITTED` 격리 수준을 기본으로 사용한다. 

JPA도 마찬가지로 보통 `READ COMITTED` 정도로 가정하고 진행하는데 동시성 문제와 같은 더 높은 격리 수준이 필요한 경우 낙관적/비관적 락을 사용하면 된다.

<br>


### 1. 낙관적(Optimistic) Lock 적용
---

낙관적 락은 트랜잭션 대부분이 충돌하지 않는다는 낙관적인 상황을 가정하는 방식이다.

한 가지 알아두어야 할 것은 낙관적 락은 **DB가 제공하는 Lock 기능**이 아닌 **JPA가 제공하는 버전 관리 기능**을 사용한다는 점이다.


JPA의 버전 관리 기능은 `@Version` 어노테이션을 통해 이루어진다.

버전 관리 기능을 적용하고자 하는 엔티티에 필드를 하나 추가하고, `@Version` 어노테이션을 붙이면 엔티티가 수정될 때마다 버전이 하나씩 증가한다.

그리고 엔티티가 수정될 때 마다 조회 시점의 버전과 수정 시점의 버전이 일치하는지를 확인한다.

만약 일치하지 않는다면 예외를 발생시킨다. 

<br>


이 부분을 프로젝트에 적용한다면, 다음과 같다.

> 1. 트랜잭션 1이 Trade(입찰) 엔티티를 조회 - version = 1
> 2. 동시에 트랜잭션 2가 Trade 엔티티를 조회 후 수정 - version = 2
> 3. 트랜잭션 1이 commit 하려는 순간 현재 자신의 version은 1인데 Trade 엔티티의 version은 2이므로 예외 발생

이렇게 version을 통해 단 1명 만이 낙찰되게끔 하는 것이 낙관적 락을 적용하는 방식이다.

그럼 이제 본격적으로 진행해보자.

<br>

#### 1-1. Trade 엔티티
---

먼저, `Trade` 엔티티에 버전 관리 기능을 적용할 필드와 어노테이션을 붙여준다.

```java
@Entity
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Builder
public class Trade extends BaseTimeEntity {

    //... 중략

    @Version
    private Integer version;
}
```

참고로 뒤에 있을 Lock 옵션을 적용하지 않아도, 엔티티에 `@Version` 어노테이션이 적용된 필드만 있어도 낙관적 락이 적용된다.


<br>

#### 1-2. 즉시 구매 로직
---

다음으로 낙관적 Lock 적용 전의 `TradeService`의 즉시 구매 로직은 다음과 같다.

구매자와 입찰을 조회해서 구매자의 주소와 포인트가 유효한지 확인하고, 입찰(Trade)의 상태 변경을 통해 낙찰되는 형식이다.

```java
@CacheEvict(value = "products",key = "#requestDto.productId")
public void immediatePurchase(String email, ImmediatePurchaseRequest requestDto) {

    User buyer = userRepository.findByEmail(email)
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.USER_NOT_FOUND));

    Trade trade = tradeRepository.findById(requestDto.getTradeId())
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.TRADE_NOT_FOUND));

    // 요청 주소가 주소록에 있는지 확인
    Address buyerAddress = buyer.getAddressList().stream()
            .filter(address -> address.getId().equals(requestDto.getAddressId()))
            .findAny()
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.ADDRESS_NOT_FOUND));

    // 구매자 포인트 충분한지 확인
    buyer.checkPointForPurchase(trade.getPrice());

    // 즉시 구매 진행 (판매자 발송 대기 상태로 변경)
    trade.registerImmediatePurchase(buyer, buyerAddress);

    // 구매자 포인트 차감
    buyer.deductPoints(trade.getPrice());

    // 구매자 포인트 차감 이력 생성 후 저장
    Point point = Point.registerPointDeductionHistory(buyer, trade.getPrice());
    pointRepository.save(point);
}
```

<br>

이 부분을 이렇게 변경했다.

```java
@CacheEvict(value = "products",key = "#requestDto.productId")
public ImmediatePurchaseResponse immediatePurchase(String email, ImmediatePurchaseRequest requestDto) {

    ImmediatePurchaseResponse response = null;

    try {
        User buyer = userRepository.findByEmail(email)
                .orElseThrow(() -> new ShoeKreamException(ErrorCode.USER_NOT_FOUND));

        Trade trade = tradeRepository.findOptimisticLockById(requestDto.getTradeId())
                .orElseThrow(() -> new ShoeKreamException(ErrorCode.TRADE_NOT_FOUND));

        response = validator.purchaseValidate(trade);
        response.setTradeId(buyer.getId(), trade.getId());

        if(!response.isEligible()) {
            log.info(response.getRejectReason());
            return response;
        }

        // 요청 주소가 주소록에 있는지 확인
        Address buyerAddress = buyer.getAddressList().stream()
                .filter(address -> address.getId().equals(requestDto.getAddressId()))
                .findAny()
                .orElseThrow(() -> new ShoeKreamException(ErrorCode.ADDRESS_NOT_FOUND));

        // 구매자 포인트 충분한지 확인
        buyer.checkPointForPurchase(trade.getPrice());

        // 구매자 포인트 차감
        buyer.deductPoints(trade.getPrice());   

        // 즉시 구매 진행 (판매자 발송 대기 상태로 변경)
        trade.registerImmediatePurchase(buyer, buyerAddress);   

        // 구매자 포인트 차감 이력 생성 후 저장
        Point point = Point.registerPointDeductionHistory(buyer, trade.getPrice());
        pointRepository.save(point);


    } catch (ObjectOptimisticLockingFailureException e) {
        return ImmediatePurchaseResponse.of(false, "이미 낙찰된 거래입니다.");
    }

    return response;
}
```

<br>

달라진 부분부터 살펴보자.


기존 `tradeRepository.findById()`가 아닌, `tradeRepository.findOptimisticLockById` 메서드로 조회해온다.

`TradeRepository` 클래스를 가보면, 다음과 같이 낙관적 Lock 옵션을 붙여주었다.

```java
@Repository
public interface TradeRepository extends JpaRepository<Trade, Long> {

    //... 중략

    @Lock(LockModeType.OPTIMISTIC)
    Optional<Trade> findOptimisticLockById(@Param("id") Long id);
}
```

<br>

기존의 void 반환이 아닌 DTO로 반환해주었다.

그 이유는 이미 낙찰된 거래인지를 확인하는 boolean 타입의 `eligible` 필드와 메세지를 출력할 `rejectReason` 필드를 생성하고 이를 반환하기 위함이다.


```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ImmediatePurchaseResponse {

    private boolean eligible;
    private String rejectReason;
    private Long tradeId;
    private Long buyerId;
    
    public void setTradeId(Long buyerId, Long tradeId) {
        this.tradeId = tradeId;
        this.buyerId = buyerId;
    }
    public ImmediatePurchaseResponse(boolean eligible, String rejectReason) {
        this.eligible = eligible;
        this.rejectReason = rejectReason;
    }

    public static ImmediatePurchaseResponse of(boolean eligible, String rejectReason) {
        return new ImmediatePurchaseResponse(eligible, rejectReason);
    }
}
```

<br>

그리고 다음과 같은 부분이 추가되었다. 

```java
@CacheEvict(value = "products",key = "#requestDto.productId")
public ImmediatePurchaseResponse immediatePurchase(String email, ImmediatePurchaseRequest requestDto) {

    //... 중략

    ImmediatePurchaseResponse response = validator.purchaseValidate(trade);

    if(!response.isEligible()) {
        log.info(response.getRejectReason());
        return response;
    }

    //... 중략
```

`TradeValidator`가 추가되었는데, 여기선 `Trade`의 상태가 입찰 신청 상태인 `PRE_OFFER`가 맞는지, 그리고 아직 구매자가 없는 상황인지를 확인한다.

입찰 신청 상태이면서 구매자가 없어야 판매 입찰이 아직 올라와 있는 상황이기 떄문에 위와 같은 상황이 아니라면 `이미 낙찰된 거래`라는 문구를 출력하게 한다.

<br>

코드는 다음과 같다.

`TradeValidator.java`

```java
@Component
public class TradeValidator {

    public ImmediatePurchaseResponse purchaseValidate(Trade trade) {
        if(!trade.getStatus().equals(TradeStatus.PRE_OFFER) || trade.getBuyer() != null) {
            return ImmediatePurchaseResponse.of(false, "이미 낙찰된 거래입니다.");
        }

        return ImmediatePurchaseResponse.of(true, null);
    }
}
```

<br>

#### 1-3. 동시성 테스트 코드
---

멀티스레드 상황을 고려한 테스트 코드이다. 좀 더 자세한 건 [여기](https://github.com/hgs-study/distributed-lock-practice)를 참고하자. 

실제로는 서로 다른 구매자의 동시 요청이 있겠지만, 테스트 상에선 서로 다른 스레드에서의 하나의 구매자로 진행했다.

같은 구매자여도 서로 다른 스레드에서는 다른 역할을 수행한다고 생각하면 된다.


```java
@SpringBootTest
@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)
@RequiredArgsConstructor
class TradeConcurrencyTest {

    private final TradeService tradeService;
    private final TradeRepository tradeRepository;
    private final UserRepository userRepository;

    User buyer;
    final ImmediatePurchaseRequest immediatePurchaseRequest
            = ImmediatePurchaseRequest.builder()
            .tradeId(1L).productId(1L).addressId(3L).build();

    @BeforeEach
    void init() {
        buyer = userRepository.findById(3L).get();
    }

    @Test
    @DisplayName("판매 입찰 1개에, 동시에 10명이 즉시 구매하려는 상황")
    void immediatePurchase() throws InterruptedException {

        final int PURCHASE_PEOPLE = 10;
        final int SALES_BID = 1;

        CountDownLatch countDownLatch = new CountDownLatch(PURCHASE_PEOPLE);

        List<ImmediateBuyer> buyers = Stream
                .generate(() -> new ImmediateBuyer(buyer,countDownLatch))
                .limit(PURCHASE_PEOPLE)
                .collect(Collectors.toList());

        buyers.forEach(buyer -> new Thread(buyer).start());
        countDownLatch.await();

        List<Trade> trades = tradeRepository.findTradeById(immediatePurchaseRequest.getTradeId());
        long counts = trades.size();

        assertThat(counts).isEqualTo(SALES_BID);
    }

    private class ImmediateBuyer implements Runnable {
        private User buyer;
        private CountDownLatch countDownLatch;

        public ImmediateBuyer(User buyer, CountDownLatch countDownLatch) {
            this.buyer = buyer;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            tradeService.immediatePurchase(buyer.getEmail(), immediatePurchaseRequest);
            countDownLatch.countDown();
        }
    }
```

동시에 10명이 즉시 구매하려는 상황이다.

`PURCHASE_PEOPLE` 이 곧 스레드의 갯수, 즉 동시에 요청하는 구매자를 의미한다.

`findTradeById()` 메서드를 통해 `SALES_BID`의 1명 만이 낙찰되는지를 확인한다.

<br>

#### 1-4. 결과 확인
---

데드락이 발생했다.

![image](https://user-images.githubusercontent.com/85394884/236520205-258c3f0b-475f-4c20-b2a1-25587fb19a25.png)

<br>

### 2. 데드락 발생 원인 파악
---

데드락, 교착 상태는 프로세스가 자원을 얻지 못하는 상태를 말한다.

여기서는 한 트랜잭션이 자원을 점유한 상태에서 다른 트랜잭션이 점유하고 있는 자원을 서로가 요구해 계속해서 자원 해제를 기다리는 상황을 말한다.


`show engine innodb status;` 명령어를 통해 History를 확인할 수 있다.

![image](https://user-images.githubusercontent.com/85394884/236686736-ed830832-7491-4974-bd44-c654105cb898.png)

<br>

히스토리를 확인해보면, 아래와 같이 `user`에서 s-Lock을 획득하고, 이후에 `user_base`에 걸린 x-Lock이 Lock 획득을 대기한다고 한다.

```MYSQL
(1) HOLDS THE LOCK(S):
RECORD LOCKS space id 1562 page no 4 n bits 80 index PRIMARY of table `shoekream-db`.`user` trx id 313014 lock mode S locks rec but not gap
```

```MYSQL
(2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 1562 page no 4 n bits 80 index PRIMARY of table `shoekream-db`.`user` trx id 313015 lock_mode X locks rec but not gap waiting
```

<br>

#### 2-1. s-lock & x-lock
---

우선, s-Lock은 공유 락(Shared Lock)이라고 하며, 읽기 잠금(Read Lock)이라고도 불린다.

한 트랜잭션에서 데이터를 읽고자 할 때 다른 트랜잭션의 s-lock은 허용이 되지만 x-lock은 불가하다.

허용이 된다는 말은 리소스에 대한 다른 트랜잭션의 `동시 읽기`는 가능하지만, 변경 중인 리소스의 `동시 읽기`는 불가능하다는 뜻과 같다.

정리하면, 어떤 리소스에 s-lock은 동시에 여러 개 적용될 수 있으나, s-lock이 하나라도 걸려있으면 x-lock은 걸 수 없다는 의미가 된다.

<br>

반면에, x-lock은 배타적 락(Exclusive Lock)이라고 하며, 쓰기 잠금(Write Lock)이라고도 불린다.

한 트랜잭션에서 데이터를 쓰고자 할 때(변경) 해당 트랜잭션이 완료될 때까지 해당 테이블 혹은 레코드(row)를 다른 트랜잭션에서 읽거나 쓰지 못하게 한다.

한 트랜잭션이 x-lock에 걸리면 그 어떤 트랜잭션도 s-lock/x-lock을 걸 수 없고, 오직 한 트랜잭션의 점유만 가능하다는 점에서 s-lock과 차이가 있다.

<br>

#### 2-2. MYSQL과 DB-Lock의 관계
---

앞서, 낙관적 락은 **DB가 제공하는 Lock 기능**이 아닌 **JPA가 제공하는 버전 관리 기능**을 사용한다고 했었다.

그런데 히스토리에는 떡하니 DB-Lock이 사용됐다고 나와있다.

<br>

[MYSQL 레퍼런스](https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html)에 다음과 같은 내용이 있다.

> If a FOREIGN KEY constraint is defined on a table, any insert, update, or delete that requires the constraint condition to be checked sets shared record-level locks on the records that it looks at to check the constraint. InnoDB also sets these locks in the case where the constraint fails.

내용인즉슨, 외래키가 있는 테이블에서, 외래키를 포함한 데이터를 `insert`, `update`, `delete` 하는 쿼리는 제약조건 확인을 위해 s-lock을 설정한다고 한다.

**즉, 판매 입찰이 낙찰되면 Trade 테이블 데이터가 update되는데 이 때 User의 id를 외래키로 가지고 있기 때문에 User 데이터에 s-lock이 걸린 것이다.**

<br>

다음으로, x-lock과 관련해선 다음과 같은 내용이 있다.

> UPDATE … WHERE … sets an exclusive next-key lock on every record the search encounters. However, only an index record lock is required for statements that lock rows using a unique index to search for a unique row.

해석하면 Update 쿼리에 사용되는 모든 레코드에 x-lock을 설정한다고 한다.

**낙찰되면서 Trade의 상태값을 변경할 때 update 쿼리가 발생하면서 x-lock이 걸린 것이다.**

<br>

결국 위의 내용을 종합해서 데드락이 발생한 상황을 정리해보면 다음과 같다.

1. 트랜잭션 A가 fk가 걸려있는 User 레코드에 s-lock을 건다.
    - User 엔티티는 UserBase 엔티티를 상속하고 있기 때문에, UserBase 레코드에도 s-lock을 건다.

2. 트랜잭션 B가 fk가 걸려있는 User 레코드에 s-lock을 건다.
    
    - 마찬가지로, User 엔티티는 UserBase 엔티티를 상속하고 있기 때문에, UserBase 레코드에도 s-lock을 건다.
    
    - s-lock은 호환 가능하므로 서로 다른 트랜잭션이 같은 레코드에 Lock을 걸 수 있다.

3. 트랜잭션 A가 즉시 구매(낙찰)되었다는 상태로 변경한다.

    - 이 때, update 쿼리가 발생하는데, User를 거쳐 UserBase 레코드에 x-lock을 걸려고 시도한다.

    - 하지만 이미 트랜잭션 B에서 UserBase 레코드에 s-lock을 걸어놨기 때문에 s-lock이 풀릴 때까지 기다린다.

4. 트랜잭션 B가 즉시 구매(낙찰)되었다는 상태로 변경한다.

    - 이 때, update 쿼리가 발생하는데, User를 거쳐 UserBase 레코드에 x-lock을 걸려고 시도한다.

    - 하지만 이미 트랜잭션 A에서 UserBase 레코드에 s-lock을 걸어놨기 때문에 s-lock이 풀릴 때까지 기다린다.


🚨 그 결과 데드락 발생!!! 🚨


낙관적 락으론 현재 상황에서 데드락을 피할 수 없다는 결론이 나왔고, 다음으로 비관적 락을 적용해서 해결하고자 한다.

<br>

### 3. 비관적(Pessimistic) Lock 적용
---

비관적 락은 낙관적 락과는 달리 트랜잭션 대부분이 충돌한다는 상황을 가정하고 우선 락을 걸고 보는 방식이다.

대표적으로 `select for update` 구문과 같은 방식이 있다.

비관적 락은 DB가 제공하는 Lock 기능을 사용하고, JPA에서는 DB가 제공하는 트랜잭션 락 메커니즘에 의존하는 방법을 제공할 뿐이다.

앞서 말했던 `select for update` 구문을 사용하면서 낙관적 락에서 사용했던 버전 정보는 사용하지 않는다.

비관적 락을 적용하면, 한 트랜잭션이 데이터를 조회한 경우, 해당 트랜잭션이 끝나기 전까지는 해당 데이터 다음의 데이터를 변경(insert/update/delete)하는 작업을 수행할 수 없게 된다.

쉽게 말하면, 첫 번째 레코드에 비관적 락이 걸리면, 두 번째 레코드에 데이터의 변경 작업은 불가능하다는 뜻이다.

<br>

이 부분을 프로젝트에 적용한다면, 다음과 같다.

> 1. 입찰 데이터에 비관적 락을 적용한다.
> 2. 그러면, 한 트랜잭션이 끝나기 전까지는 즉, 입찰이 낙찰될 때 까지는 어떤 트랜잭션도 대기만 해야 한다.

직접 적용해보자.

<br>

#### 3-1. 변경 사항
---

우선, `Trade` 엔티티의 버전 정보는 이제 사용하지 않으므로 제거한다.

<br>

다음으로, `TradeRepository`의 `tradeRepository.findOptimisticLockById` 메서드를 적절한 이름으로 변경하고, 옵션을 `@Lock(LockModeType.PESSIMISTIC_WRITE)`으로 변경한다.

```java
@Repository
public interface TradeRepository extends JpaRepository<Trade, Long> {

    //... 중략

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<Trade> findPessimisticLockById(@Param("id") Long id);
}
```

<br>

`TradeService`의 즉시 구매 로직은 크게 달라진 부분은 없고, 위에서 만든 `findPessimisticLockById()` 메서드로 변경만 해주면 된다.

```java
@CacheEvict(value = "products",key = "#requestDto.productId")
public ImmediatePurchaseResponse immediatePurchase(String email, ImmediatePurchaseRequest requestDto) {

    User buyer = userRepository.findByEmail(email)
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.USER_NOT_FOUND));

    Trade trade = tradeRepository.findPessimisticLockById(requestDto.getTradeId())
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.TRADE_NOT_FOUND));

    ImmediatePurchaseResponse response = validator.purchaseValidate(trade);
    response.setTradeId(buyer.getId(), trade.getId());

    if(!response.isEligible()) {
        log.info(response.getRejectReason());
        return response;
    }

    // 요청 주소가 주소록에 있는지 확인
    Address buyerAddress = buyer.getAddressList().stream()
            .filter(address -> address.getId().equals(requestDto.getAddressId()))
            .findAny()
            .orElseThrow(() -> new ShoeKreamException(ErrorCode.ADDRESS_NOT_FOUND));

    // 구매자 포인트 충분한지 확인
    buyer.checkPointForPurchase(trade.getPrice());

    // 구매자 포인트 차감
    buyer.deductPoints(trade.getPrice());

    // 즉시 구매 진행 (판매자 발송 대기 상태로 변경)
    trade.registerImmediatePurchase(buyer, buyerAddress);

    // 구매자 포인트 차감 이력 생성 후 저장
    Point point = Point.registerPointDeductionHistory(buyer, trade.getPrice());
    pointRepository.save(point);

    return response;
}
```

<br>

동시성 테스트 코드에도 변경사항은 없다. 동시에 10명이 즉시 구매하려는 상황을 가정한다.

```java
@SpringBootTest
@TestConstructor(autowireMode = TestConstructor.AutowireMode.ALL)
@RequiredArgsConstructor
class TradeConcurrencyTest {

    private final TradeService tradeService;
    private final TradeRepository tradeRepository;
    private final UserRepository userRepository;

    User buyer;
    final ImmediatePurchaseRequest immediatePurchaseRequest
            = ImmediatePurchaseRequest.builder()
            .tradeId(1L).productId(1L).addressId(3L).build();

    @BeforeEach
    void init() {
        buyer = userRepository.findById(3L).get();
    }

    @Test
    @DisplayName("판매 입찰 1개에, 동시에 10명이 즉시 구매하려는 상황")
    void immediatePurchase() throws InterruptedException {

        final int PURCHASE_PEOPLE = 10;
        final int SALES_BID = 1;

        CountDownLatch countDownLatch = new CountDownLatch(PURCHASE_PEOPLE);

        List<ImmediateBuyer> buyers = Stream
                .generate(() -> new ImmediateBuyer(buyer,countDownLatch))
                .limit(PURCHASE_PEOPLE)
                .collect(Collectors.toList());

        buyers.forEach(buyer -> new Thread(buyer).start());
        countDownLatch.await();

        List<Trade> trades = tradeRepository.findTradeById(immediatePurchaseRequest.getTradeId());
        long counts = trades.size();

        assertThat(counts).isEqualTo(SALES_BID);
    }

    private class ImmediateBuyer implements Runnable {
        private User buyer;
        private CountDownLatch countDownLatch;

        public ImmediateBuyer(User buyer, CountDownLatch countDownLatch) {
            this.buyer = buyer;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            tradeService.immediatePurchase(buyer.getEmail(), immediatePurchaseRequest);
            countDownLatch.countDown();
        }
    }
```

<br>

#### 3-2. 결과 확인
---

낙관적 락을 적용했을 때와는 달리 1개의 낙찰자가 생기고, 그 외 9명은 `이미 낙찰된 거래입니다.` 라는 의도한 메시지가 출력되었다.

![image](https://user-images.githubusercontent.com/85394884/236697980-01bb1e3b-31e7-40ef-b74e-82789ee3988d.png)

<br>

그리고, 다음과 같이 `select for update` 쿼리가 발생한 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/85394884/236698160-69abed4b-2c5b-4ed0-8e03-b3387010ad42.png)

<br>


#### 3-3. 비관적 락과 데드락의 관계
---

현재, 판매 입찰 & 즉시 구매 상황에선 Trade 테이블의 특정 레코드에만 Lock을 걸고 진행했기 때문에 데드락 상황이 발생하지 않았다.

하지만, 비관적 락도 데드락 상황이 발생할 수 있다.

예를 들어 여러 테이블에 Lock을 걸거나, 한 테이블의 여러 레코드에 Lock을 거는 상황에선 데드락이 발생할 수도 있다.

그리고 추가적으로 비관적 락은 모든 트랜잭션에 락을 걸기 때문에 필요하지 않은 상황에서도 락이 걸린다.

때문에 데이터 정합성 보장에는 좋을지 몰라도 성능 상의 문제가 있어 엄청나게 많은 요청이 들어온다면 처리하기 힘들 수 있다.


<br>

### 4. 결과 정리
---

동시성 문제가 발생할 부분에 낙관적 락, 비관적 락을 순차적으로 적용해보았다.

낙관적 락으로는 데드락이 발생하기 때문에 해결할 수 없었고, 비관적 락을 통해서야 비로소 해결 가능했다.

중요한 것은 낙관적 락이든 비관적 락이든 단일 DB 환경에서만 적용 가능하다.

Master/Slave와 같은 분산 DB 환경에선 다른 방법으로 해결해야 한다.

그 방법은 다음 게시물에서 진행해보겠다.


<br>


## References

* [동시성 문제 해결하기 V2 - 비관적 락(Pessimistic Lock)](https://velog.io/@znftm97/%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-V2-%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BDPessimistic-Lock)
* [데이터베이스 - Exclusive lock과 Shared lock의 차이](https://jeong-pro.tistory.com/94)