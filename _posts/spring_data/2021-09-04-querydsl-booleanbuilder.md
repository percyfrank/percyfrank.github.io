---
title: "QueryDSL BooleanExpression을 통한 동적 쿼리 생성"
excerpt: "다양한 조건에 해당하는 구문을 직관적이고 재사용성 있게 작성해보자."
categories:
  - Spring Data
tags:
  - Spring Data
date: 2021-09-04
last_modified_at: 2021-09-04
---

## 1. BooleanBuilder

> PostCustomRepositoryImpl.java

```java
@Override
public List<Post> findPostsOrderByDateDesc(
    Pageable pageable,
    SearchCondition searchCondition
) {
    return selectPostInnerFetchJoinUser()
        .where(isActivePostUnderSearchCondition(searchCondition))
        .orderBy(QPOST.baseDate.createdDate.desc())
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();
}

private Predicate isActivePostUnderSearchCondition(SearchCondition searchCondition) {
    BooleanBuilder booleanBuilder = new BooleanBuilder();
    String keyword = searchCondition.getKeyword();
    booleanBuilder.and(isActivePost());
    if (searchCondition.isForTitle()) {
        booleanBuilder.and(QPOST.postContent.title.containsIgnoreCase(keyword));
    }
    if (searchCondition.isForName()) {
        booleanBuilder.and(QPOST.user.name.containsIgnoreCase(keyword));
    }
    if (searchCondition.isForContent()) {
        booleanBuilder.and(QPOST.postContent.content.containsIgnoreCase(keyword));
    }
    return booleanBuilder;
}
```

<br>

## 2. BooleanExpression

> PostCustomRepositoryImpl.java

```java
@Override
public List<Post> findPostsOrderByDateDesc(
    Pageable pageable,
    SearchCondition searchCondition
) {
    return selectPostInnerFetchJoinUser()
        .where(eqActivePost(),
            eqTitle(searchCondition),
            eqName(searchCondition),
            eqContent(searchCondition))
        .orderBy(QPOST.baseDate.createdDate.desc())
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();
}

public BooleanExpression eqTitle(SearchCondition searchCondition) {
    if (!searchCondition.isForTitle()) {
        return null;
    }
    return QPOST.postContent.title.containsIgnoreCase(searchCondition.getTitle());
}

private BooleanExpression eqContent(SearchCondition searchCondition) {
    if (!searchCondition.isForContent()) {
        return null;
    }
    return QPOST.postContent.title.containsIgnoreCase(searchCondition.getContent());
}

private BooleanExpression eqName(SearchCondition searchCondition) {
    if (!searchCondition.isForName()) {
        return null;
    }
    return QPOST.postContent.title.containsIgnoreCase(searchCondition.getName());
}
```

* 예제에 사용하기 적합한 코드는 아니지만, 아래의 코드처럼 리팩토링이 가능해보인다.

> Reference 참고 코드

```java
public List<Study> findLiveStudyBySearch(String title, Integer bigCity, Integer smallCity) {
    return jpaQueryFactory.selectFrom(study)
                          .where(eqTitle(title),
                                 eqBigCity(bigCity),
                                 eqSmallCity(smallCity),
                                 eqDeleted(false))
                          .fetch();
}

private BooleanExpression eqTitle(String title) {
    if (StringUtils.isBlank(title)) {
        return null;
    }
    return study.title.containsIgnoreCase(title);
}

private BooleanExpression eqBigCity(Integer bigcity) {
    if (bigcity != null) {
        return null;
    }
    return study.bigcity.eq(bigcity);
}

private BooleanExpression eqSmallCity(Integer smallCity)
    if (smallCity != null) {
        return null;
    }
    return study.smallCity.eq(smallCity);
}

private BooleanExpression eqDeleted(boolean deleted) {
    return study.deleted.eq(deleted);
}
```

* Reference의 코드가 조금 더 참고하기 적합한 코드라고 생각된다.

<br>

---

## References

* [[QueryDSL] BooleanExpression을 통한 복잡한 동적쿼리 표현(BooleanBuilder 제거)](https://sas-study.tistory.com/393)
