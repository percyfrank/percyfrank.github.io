---
title: "JPA N + 1 문제 및 해결 방법"
excerpt: "2개 이상의 컬렉션을 즉시 로딩하는 방법이 없을까?"
categories:
  - Spring Data
tags:
  - Spring Data
date: 2021-07-25
last_modified_at: 2021-07-25
---

> [실습 Repository](https://github.com/xlffm3/springboot-jpa-learning-test)

## 1. N + 1 문제 발생

> PostRepositoryTest.java

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class PostRepositoryTest {

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private TestEntityManager testEntityManager;

    @BeforeEach
    void setUp() {
        Post post = new Post("hello this is jpa test!");
        Comment comment = new Comment("hi this is amazing!");
        Tag tag = new Tag("abc");
        post.addComment(comment);
        post.addTag(tag);

        Post post2 = new Post("hello this is another post!");
        Comment comment2 = new Comment("hi this is another comment!");
        Tag tag2 = new Tag("def");
        post2.addComment(comment2);
        post2.addTag(tag2);

        Post post3 = new Post("hello this is another post!");
        Comment comment3 = new Comment("hi this is another comment!");
        Tag tag3 = new Tag("gka");
        post3.addComment(comment3);
        post3.addTag(tag3);

        postRepository.save(post);
        postRepository.save(post2);
        postRepository.save(post3);

        testEntityManager.flush();
        testEntityManager.clear();
    }

    @DisplayName("글로벌 지연 전략으로 인해 N + 1 쿼리가 발생한다.")
    @Test
    void findPostsCausingNPlusOne() {
        System.out.println("========= find =========\n\n\n");
        List<Post> posts = postRepository.findAll();

        assertThat(posts).hasSize(3);

        for (Post p : posts) {
            System.out.println(p.getComments().get(0).getContent());
        }

        assertThat(posts).hasSize(3);
    }
}
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_1_,
        post0_.content as content2_1_
    from
        post post0_
Hibernate:
    select
        comments0_.post_id as post_id3_0_0_,
        comments0_.id as id1_0_0_,
        comments0_.id as id1_0_1_,
        comments0_.content as content2_0_1_,
        comments0_.post_id as post_id3_0_1_
    from
        comment comments0_
    where
        comments0_.post_id=?
hi this is amazing!
Hibernate:
    select
        comments0_.post_id as post_id3_0_0_,
        comments0_.id as id1_0_0_,
        comments0_.id as id1_0_1_,
        comments0_.content as content2_0_1_,
        comments0_.post_id as post_id3_0_1_
    from
        comment comments0_
    where
        comments0_.post_id=?
hi this is another comment!
Hibernate:
    select
        comments0_.post_id as post_id3_0_0_,
        comments0_.id as id1_0_0_,
        comments0_.id as id1_0_1_,
        comments0_.content as content2_0_1_,
        comments0_.post_id as post_id3_0_1_
    from
        comment comments0_
    where
        comments0_.post_id=?
hi this is another comment!
```

포스트 전체 조회 쿼리 1개 + 반복문 돌면서 지연 로딩되는 댓글 조회 쿼리 N개가 발생한다. 포스트가 10만개라면 10만개의 쿼리가 발생한다.

* 연관 관계의 글로벌 로딩 전략이 즉시 로딩이더라도 JPQL을 사용하거나 ``findAll()`` 메서드를 호출하면 결국 N + 1 문제가 발생한다.
* 전체 조회 쿼리가 나간 뒤, 각 엔티티별로 즉시 로딩이 필요한 필드에 대해 추가적인 쿼리가 나가기 때문이다.

<br>

## 2. Fetch Join

> PostRepository.java

```java
@Query("select p from Post p join fetch p.comments")
List<Post> findAllInnerFetchJoin();
```

> PostRepositoryTest.java

```java
@DisplayName("일반적인 Fetch Join 사용시 내부 조인이 발생한다.")
@Test
void findPostsUsingInnerFetchJoin() {
    System.out.println("========= find =========\n\n\n");
    postRepository.save(new Post("dummy post"));
    List<Post> posts = postRepository.findAllInnerFetchJoin();

    assertThat(posts).hasSize(3);

    for (Post p : posts) {
        System.out.println(p.getComments().get(0).getContent());
    }
}
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    inner join
        comment comments1_
            on post0_.id=comments1_.post_id
hi this is amazing!
hi this is another comment!
hi this is another comment!
```

Fetch Join을 통해 연관 엔티티들을 한꺼번에 조회할 수 있다.

* 특별하게 명시하지 않으면 Fetch Join은 기본적으로 Inner Join이라는 점을 참고하자.
* 따라서 Comment가 전혀 없는 Post는 조회 결과 누락된다.

> PostRepository.java

```java
@Query("select p from Post p left join fetch p.comments")
List<Post> findAllOuterFetchJoin();
```

> PostRepositoryTest.java

```java
@DisplayName("명시함으로써 Fetch Join에 외부 조인이 발생한다.")
@Test
void findPostsUsingOuterFetchJoin() {
    System.out.println("========= find =========\n\n\n");
    postRepository.save(new Post("dummy post"));
    List<Post> posts = postRepository.findAllOuterFetchJoin();

    assertThat(posts).hasSize(4);
    assertThatCode(() -> {
        for (Post p : posts) {
            System.out.println(p.getComments().get(0).getContent());
        }
    }).isInstanceOf(IndexOutOfBoundsException.class);
}
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    left outer join
        comment comments1_
            on post0_.id=comments1_.post_id
hi this is amazing!
hi this is another comment!
hi this is another comment!
```

* 명시하면 Fetch Join도 Outer로 걸 수 있다.
* Comment가 없는 Post도 조회 결과에 포함된다.

<br>

## 3. @EntityGraph

> PostRepository.java

```java
@EntityGraph(attributePaths = "comments")
@Query("select p from Post p")
List<Post> findAllEntityGraph();

@EntityGraph(attributePaths = {"comments", "comments.like"})
@Query("select p from Post p")
List<Post> findAllEntityGraphWithSubGraph();
```

> PostRepositoryTest.java

```java
@DisplayName("EntityGraph를 통해 연관 엔티티들을 즉시 로딩한다.")
@Test
void findPostsUsingEntityGraph() {
    System.out.println("========= find =========\n\n\n");
    postRepository.save(new Post("dummy post"));
    List<Post> posts = postRepository.findAllEntityGraph();

    assertThat(posts).hasSize(4);
    assertThatCode(() -> {
        for (Post p : posts) {
            System.out.println(p.getComments().get(0).getContent());
        }
    }).isInstanceOf(IndexOutOfBoundsException.class);
}

@DisplayName("EntityGraph와 Subgraph를 통해 연관 엔티티들을 즉시 로딩한다.")
@Test
void findPostsUsingEntityGraphWithSubGraph() {
    System.out.println("========= find =========\n\n\n");
    postRepository.save(new Post("dummy post"));
    List<Post> posts = postRepository.findAllEntityGraphWithSubGraph();

    assertThat(posts).hasSize(4);
    assertThatCode(() -> {
        for (Post p : posts) {
            System.out.println(p.getComments().get(0).getContent());
        }
    }).isInstanceOf(IndexOutOfBoundsException.class);
}
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    left outer join
        comment comments1_
            on post0_.id=comments1_.post_id
hi this is amazing!
hi this is another comment!
hi this is another comment!

Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        like2_.id as id1_3_2_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    left outer join
        comment comments1_
            on post0_.id=comments1_.post_id
    left outer join
        likes like2_
            on comments1_.like_id=like2_.id
hi this is amazing!
hi this is another comment!
hi this is another comment!
```

* @EntityGraph를 사용함으로써 연관 엔티티들을 즉시 로딩으로 조회할 수 있다.
* 연관 엔티티와 해당 연관 엔티티의 연관 엔티티까지 다양하게 조회가 가능하다.
* 자동으로 외부 조인이 발생됨을 참고하자.

### 3.1. @NamedEntityGraphs

@NamedEntityGraphs 또한 @EntityGraph와 동일한 효과를 낼 수 있다.

* @NamedEntityGraphs는 Entity 클래스에 수 많은 설정 코드를 요구한다.
* 특정 로직에서는 특정 Fetch 전략을 수행한다는 것은 Entity의 책임이 아니다.
  * 유동적인 Fetch 전략은 전적으로 Service 혹은 Repository가 수행해야 한다.

<br>

## 4. Cartesian Product

> PostRepositoryTest.java

```java
@DisplayName("일반적인 Fetch Join으로 조회시 카테시안 곱이 발생한다.")
@Test
void findCartesianProduct() {
    postRepository.deleteAll();

    for (int i = 0; i < 10; i++) {
        Post post = new Post("dummy post");
        Comment comment1 = new Comment("hi");
        Comment comment2 = new Comment("hi2");
        post.addComment(comment1);
        post.addComment(comment2);
        postRepository.save(post);
    }

    testEntityManager.flush();
    testEntityManager.clear();

    List<Post> posts = postRepository.findAllInnerFetchJoin();

    assertThat(posts).hasSize(20);
}
```

* ToOne의 관계에서 Fetch Join 혹은 @EntityGraph를 사용할 때는 문제가 없으나, ToMany의 경우 카테시안 곱이 발생한다.

> PostRepository.java

```java
@Query("select distinct p from Post p join fetch p.comments")
List<Post> findAllInnerFetchJoinWithDistinct();
```

> PostRepositoryTest.java

```java
@DisplayName("distinct 키워드를 통해 컬렉션 Fetch Join 중복을 제거한다.")
@Test
void removeCartesianProduct() {
    postRepository.deleteAll();

    for (int i = 0; i < 10; i++) {
        Post post = new Post("dummy post");
        Comment comment1 = new Comment("hi");
        Comment comment2 = new Comment("hi2");
        post.addComment(comment1);
        post.addComment(comment2);
        postRepository.save(post);
    }

    testEntityManager.flush();
    testEntityManager.clear();

    List<Post> posts = postRepository.findAllInnerFetchJoinWithDistinct();

    assertThat(posts).hasSize(10);
}
```

* distinct 키워드를 통해 엔티티 중복을 제거한다.
* 혹은 컬렉션을 Set으로 설정하는 방법도 고려할 수 있다.

<br>

## 5. Batch

> PostRepositoryTest.java

```java
@DisplayName("2개 이상의 컬렉션은 Fetch Join할 수 없다.")
@Test
void cannotFetchJoinMoreThanTwoCollections() {
    EntityManager entityManager = testEntityManager.getEntityManager();

    assertThatCode(() -> {
        entityManager.createQuery("select p from Post p join fetch p.comments join fetch p.tags", Post.class);
    }).hasRootCauseInstanceOf(MultipleBagFetchException.class);
}
```

카테시안 곱 이슈로 인해 JPA는 한 번에 2개 이상의 컬렉션을 Fetch Join할 수 없다.

* MultipleBagFetchException이 발생한다.
* 컬렉션이 Set이라면 가능하다.

> PostRepository.java

```java
@Query("select p from Post p join fetch p.comments")
List<Post> findAllPagingWithFetchJoin(Pageable pageable);
```

> PostRepositoryTest.java

```java
@DisplayName("컬렉션을 Fetch Join하는 경우 페이징시 경고가 발생한다.")
@Test
void findPagingWithFetchJoin() {
    postRepository.findAllPagingWithFetchJoin(PageRequest.of(0, 2));
}
```

> SQL

```sql
2021-07-25 20:42:42.790  WARN 2456 --- [    Test worker] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    inner join
        comment comments1_
            on post0_.id=comments1_.post_id
```

1:N 관계를 Fetch Join하면 몇 개의 Row까지 가지고 와야 하는지 예측할 수 없어, 전체 데이터를 조회한 다음 어플리케이션단(메모리)에서 페이지네이션 작업을 수행한다. OutOfMemory 에러가 발생할 수 있다.

* 컬렉션을 Fetch Join할 때 페이징 API를 사용하면 경고 로그가 발생한다.
* 실제로 날아간 SQL을 보면 별도의 LIMIT나 OFFSET이 없다.

> application.properties

```properties
spring.jpa.properties.hibernate.default_batch_fetch_size=1000
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_4_,
        post0_.content as content2_4_,
        post0_.created_at as created_3_4_,
        post0_.github_repo_url as github_r4_4_,
        post0_.updated_at as updated_5_4_,
        post0_.user_id as user_id6_4_
    from
        post post0_
    order by
        post0_.id limit ? offset ?
Hibernate:
    select
        posttags0_.post_id as post_id2_5_1_,
        posttags0_.id as id1_5_1_,
        posttags0_.id as id1_5_0_,
        posttags0_.post_id as post_id2_5_0_,
        posttags0_.tag_id as tag_id3_5_0_
    from
        post_tag posttags0_
    where
        posttags0_.post_id in (
            ?, ?
        )
Hibernate:
    select
        tag0_.id as id1_6_0_,
        tag0_.name as name2_6_0_
    from
        tag tag0_
    where
        tag0_.id in (
            ?, ?, ?, ?, ?
        )
Hibernate:
    select
        comments0_.post_id as post_id3_0_1_,
        comments0_.id as id1_0_1_,
        comments0_.id as id1_0_0_,
        comments0_.content as content2_0_0_,
        comments0_.post_id as post_id3_0_0_,
        comments0_.user_id as user_id4_0_0_
    from
        comment comments0_
    where
        comments0_.post_id in (
            ?, ?
        )
```

**Fetch Join해야 할 ToMany 관계의 컬렉션이 2개 이상일 때, 혹은 페이징 API를 쓸 때 성능을 개선하고 싶다면 Batch를 적용한다.**

* 기본적으로 N + 1 문제는 여러 부모 엔티티의 ID를 가지고 자식 엔티티를 조회하기 때문에, ``부모 엔티티의 개수 * 지연 로딩할 자식 엔티티 개수``만큼 추가 쿼리가 나간다.
* 부모 엔티티 N개의 ID를 가지고 자식 엔티티에게서 N번 조건 질의를 하기 보다는, 명시된 크기 만큼 부모 엔티티 ID를 In절로 묶어 조회한다.

<br>

## 6. 기타

> Article.java

```java
@Entity
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.EAGER, cascade = CascadeType.PERSIST)
    @JoinColumn(name = "category_id")
    private Category category;

    @ManyToOne(fetch = FetchType.EAGER, optional = false, cascade = CascadeType.PERSIST)
    @JoinColumn(name = "subject_id")
    private Subject subject;

    @ManyToOne(fetch = FetchType.EAGER, cascade = CascadeType.PERSIST)
    @JoinColumn(name = "writer_id", nullable = false)
    private Writer writer;
}
```

* Article과 N:1 관계를 맺은 엔티티들에 대해 optional 혹은 nullable 옵션 여하에 따라 어떻게 즉시 로딩이 달라질까?

> SQL

```sql
Hibernate:
    select
        article0_.id as id1_0_0_,
        article0_.category_id as category2_0_0_,
        article0_.subject_id as subject_3_0_0_,
        article0_.writer_id as writer_i4_0_0_,
        category1_.id as id1_1_1_,
        subject2_.id as id1_5_2_,
        writer3_.id as id1_7_3_
    from
        article article0_
    left outer join
        category category1_
            on article0_.category_id=category1_.id
    inner join
        subject subject2_
            on article0_.subject_id=subject2_.id
    inner join
        writer writer3_
            on article0_.writer_id=writer3_.id
    where
        article0_.id=?
```

optional 혹은 nullable이 false로 되어있으면 Inner Join이 발생하고 그 외의 경우 Outer Join이 발생한다.

* 해당 칼럼에 null이 들어가지 않으니 Inner Join으로도 데이터 누락이 발생하지 않기 때문이다.
* 글로벌 즉시 로딩 전략과 DDL이 어떻든간에, JPQL의 Fetch Join을 사용하면 별도로 명시하지 않으면 항상 Inner Join이 기본이다.

### 6.1. 1:N의 즉시 로딩

> PostRepository.java

```java
@Query("select p from Post p join fetch p.comments where p.id = :id")
Optional<Post> findByIdWithInnerJoin(@Param("id") Long id);
```

> PostRepositoryTest.java

```java
@DisplayName("inner join일 때 우측 데이터에 값이 없으면 찾을 수 없다.")
@Test
void cannotFindPost() {
    postRepository.deleteAll();
    Post save = postRepository.save(new Post("hi"));

    testEntityManager.flush();
    testEntityManager.clear();

    Optional<Post> post = postRepository.findByIdWithInnerJoin(save.getId());

    assertThat(post).isEmpty();
}
```

> SQL

```sql
Hibernate:
    select
        post0_.id as id1_4_0_,
        comments1_.id as id1_2_1_,
        post0_.content as content2_4_0_,
        comments1_.content as content2_2_1_,
        comments1_.like_id as like_id3_2_1_,
        comments1_.post_id as post_id4_2_1_,
        comments1_.post_id as post_id4_2_0__,
        comments1_.id as id1_2_0__
    from
        post post0_
    inner join
        comment comments1_
            on post0_.id=comments1_.post_id
    where
        post0_.id=?
```

그렇다면 왜 @OneToMany 연관 관계 컬렉션의 글로벌 즉시 로딩 전략은 항상 기본적으로 Outer Join으로 잡힐까? 예제는 Post 1개를 조회할 때 1:N 관계의 모든 Comment를 함께 조회하도록 Fetch Join을 사용했다. 별도로 명시하지 않아 Inner Join이 잡힌다.

SQL 질의 결과를 바탕으로 Post 엔티티 인스턴스 등을 구성하는데, 댓글이 없는 Post의 경우 Inner Join시 최종적으로 아무런 결과가 조회되지 않는다. Comment 테이블에 해당 POST_ID를 참조하는 Row가 없어, POST_ID가 존재하더라도 결과적으로 SELECT 절 조회 결과에서는 누락된다.

Fetch Join과 같은 특수한 상황이 아닌 @EntityGraph나 글로벌 즉시 로딩 전략을 사용하는 경우, 이러한 연유로 1:N 관계 컬렉션을 기본적으로 Outer Join으로 조회하는게 아닐까?

<br>

---

## References

* [JPA N+1 문제 및 해결방안](https://jojoldu.tistory.com/165)
* [MultipleBagFetchException 발생시 해결 방법](https://jojoldu.tistory.com/457)
