---
title: "[Java ORM 표준 JPA 프로그래밍] 2장 : JPA 시작하기"
excerpt: "Inflearn 김영한님 강의를 참고하여 정리한 필기입니다."
categories:
  - Spring Data
tags:
  - Spring Data
  - Java ORM 표준 JPA 프로그래밍
date: 2021-06-18
last_modified_at: 2021-06-18
---

## 1. JPA 설정

> Persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             version="2.2" xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <class>practice.docs.spring.domain.Member</class>
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create"/>-->
        </properties>
    </persistence-unit>
</persistence>
```

* Spring Data Jpa가 아닌 Hibernate만 사용할 때 위의 설정을 추가한다.
* Gradle을 사용하는 경우 Entity를 인식하지 못하는 경우가 있어 맵핑 클래스를 ``<class>practice.docs.spring.domain.Member</class>``로 지정해준다.
* ``classpath:/META-INF/``에 위치시킨다.
* javax.persistence : JPA 표준 속성이다.
* hibernate로 시작 : 하이버네이트 전용 속성이다.
* JPA는 특정 데이터베이스에 종속되지 않지만, 각 DB는 제공하는 SQL 문법이 다소 다르다.
  * 방언 : SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능이다.
  * 방언 설정을 지정해두면 JPA가 쿼리 작업을 현재 DB 방언에 적절한 쿼리문으로 변환해준다.

<br>

## 2. 예시

![image](https://user-images.githubusercontent.com/56240505/122572364-a35f6600-d088-11eb-853b-1a95492d6b87.png)

* 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유한다.
* 엔티티 매니저는 쓰레드간에 공유하지 않고 사용하면 버린다.
  * 내부적으로 커넥션을 사용하기에 사용하면 항상 ``close()``로 리소스를 반환한다.
* JPA의 모든 데이터 변경은 트랜잭션 안에서 실행되어야 한다.

> Member.java

```java
@Entity
public class Member {

    @Id
    private Long id;
    private String name;

    //getter, setter
}
```

> Application.java

```java
public class Application {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        EntityManager entityManager = emf.createEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();
        try {
//            Member member = new Member();
//            member.setId(1L);
//            member.setName("abc");
//            entityManager.persist(member);

//            Member member = entityManager.find(Member.class, 1L);
//            assert member.getName().equals("abc");
//
//            member.setName("Hello JPA");
//            Member changedMember = entityManager.find(Member.class, 1L);
//            assert changedMember.getName().equals("Hello JPA");

            List<Member> resultList = entityManager.createQuery("select m from Member as m", Member.class)
                    .setFirstResult(5)
                    .setMaxResults(10) //페이징
                    .getResultList();
            transaction.commit();
        } catch (RuntimeException e) {
            transaction.rollback();
        } finally {
            entityManager.close();
        }
        emf.close();
    }
}
```

* EntityManager를 통해 CRUD를 진행할 수 있으며, 엔티티를 수정하면 별도의 update 메서드 없이 커밋할 때 수정 내용이 반영된다.
  * 더티 체킹을 사용하기 때문이다.
* JPA를 사용하면 엔티티 객체를 중심으로 개발할 수 있으나, 검색 쿼리가 문제다.
* 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색한다.
  * 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
* 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.
  * 다시금 DB에 종속될 수 있는 여지가 있다.

### 2.1. JPQL

* JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다.
* SQL과 문법이 유사하다.
* JPQL은 엔티티 객체를 대상으로 하는 쿼리라면, SQL은 데이터베이스 테이블을 대상으로 하는 쿼리다.
  * ``select m from Member as m``과 같이 쿼리 결과를 테이블이 아닌 엔티티 객체로 만들기 위해 다소 특별한 문법을 사용한다.

<br>

---

## References

*	자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한, Inflearn)
