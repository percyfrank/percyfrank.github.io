> 나중에 정리할 내용들 시작

* 검색엔진 변경


## transactional?

* 문제 https://mommoo.tistory.com/92

CSRF ??
 나중에 정리할 내용 종료


 @Enumerated


 jsonvalue notnull default
 @JsonIgnore
 테스트깨질때... 값이 null인데 json 반환안

 ## 페이징

 * Pageable 인터페이스를 service단에서 사용해서 repository에서 받아옴
 * @PageDefualt
 * 적용시 test 코드의 jsonpage 변수명 변경

 ## 페이징
 sql절이 베이스긴함
 @PageableDefault(세부 옵션) Pageable Pageable;
 Pagnation 클래스 정의 (총 토탈 페이지가 몇개인지 현재 어디인지 등)
 이를 Header.ok(Users).ok(Paganation) 등 빌더를 통해 연속으로 넣어서 줌



2강 주로 JPA

## ToStringExclude (JPA)

* 롬복에서 toString EqualsAndHashcode

## 관계 맵핑 OneToOne

cascade persist(영속성 함께 관리) + cascade merge
,cascade remove
 orphanRemoval
 FetchType
Optional

## QUERY 메소드
키워드 검색 ㄱ

##Embded, Embeddable
@Valid, @Query

* Embded == birthday 객체 생성 monthOf로 더 세밀하게 가능.

## data.sql
resource 하위

## GenerationType=identity
pk unique key

 @Lombok 코드
 @Slf4j 로깅 가능


## json list에 대한 정규식

일대일
일대다
다대일
다대다

다대다의 경우 더 나은 쿼리를 위해

1:N N:1이 되도록 중간에 객체 추가

FetchType Lazy 지연로딩 Eager 즉시로딩(연관관계 설정된 엔티티에 대해 여러 테이블의 정보까지 조인
  Eager는 주로 1:1일때.. 많은 연관관계 있을때는 Lazy를 주로)


## mappedBy 연관관계

자기자신일 경우 tostring 오버플로나니 @ToString(exclude == 'xx')


## EnableJpaAudting
entityListener 설정.
자동으로 날짜들이 CreatedBy, ModifiedBy 등이 동기화 저장됨.
CreatedDate

## jsonProperty("캐멀케이스 등")으로 제이슨 이름명 설정
* 혹은 spring.jackson.property-naming:SNAKE_CASE 등
