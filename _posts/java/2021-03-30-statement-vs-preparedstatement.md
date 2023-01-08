---
title: "Statement vs PreparedStatement"
excerpt: "PreparedStatement는 캐시를 사용함으로써 성능을 향상 시킨다."
categories:
  - Java
tags:
  - Java
date: 2021-03-30
last_modified_at: 2021-03-30
---

## 1. Statement

Statement를 사용하는 경우 아래 세 가지 단계를 항상 수행한다.

* 쿼리 문장 분석
* 컴파일
* 실행

Statement의 특징은 다음과 같다.

* 코드의 가독성이 좋지 못하고, SQL Injection 공격에 취약하다.
* 최적화의 여지가 없으며 캐시를 사용할 수 없다.
* 배치 업데이트 또한 개별적으로 수행된다.
* 파일 및 배열 등을 저장하거나 회수하는데 사용할 수 없다.
* DDL 쿼리 작성에는 적합하다.

<br>

## 2. PreparedStatement

PreparedStatement는 쿼리를 처음 사용할 때 컴파일하고 캐시에 담아 둔다. 동일한 쿼리를 반복 사용하는 경우 캐시에서 재사용하기 때문에, Statement보다 성능이 우수하다.

* 가독성이 우수하며, SQL Injection 공격에 상대적으로 안전하다.
* 단일 DB 커넥션으로도 배치 작업을 수행할 수 있다.
* BLOB 및 CLOB 등을 통해 파일을 저장하거나 회수하는데 사용할 수 있다.

<br>

---

## References

* [Difference Between Statement and PreparedStatement](https://www.baeldung.com/java-statement-preparedstatement)
