---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-api/"}
---

JPQL은 **엔티티 객체를 대상으로 쿼리**하는 객체 지향 쿼리 언어다. 또한 SQL을 추상화 했기 때문에 **특정 데이터베이스에 의존하지 않는다.**

JPQL은 매핑 정보, 방언 등이 조합돼서 **결국 SQL로 변환**된다.

## JPQL - 기본 문법

### 기본 문법

```java
SELECT m from Member m WHERE m.age > 18
```

엔티티와 속성은 대소문자를 구분한다.
- Member
- age

JPQL 키워드는 대소문자를 구분하지 않는다.
- SELECT
- from
- WHERE

테이블 이름이 아닌 엔티티 이름을 사용한다.
- MEMBER (x) -> Member (o)

별칭은 필수로 작성해야 한다.
- Member (as) m

### 집합과 정렬

```java
select
	count(m),     // 회원 수
	sum(m.age),   // 나이 합
	avg(m.age),   // 평균 나이
	max(m.age),   // 연장자
	min(m.age)    // 연소자
from Member m
```

`group by`, `having`, `order by` 전부 원래 사용하던 것처럼 사용하면 됨


### TypeQuery, Query

TypeQuery: 반환 타입이 명확할 때 사용한다.
Query: 반환 타입이 명확하지 않을 때 사용한다.

```java
TypeQuery<Member> typeQuery = em.createQuery("select m from Member m", Member.class);
Query query = em.createQuery("select m.username, m.age from Member m");
```


### 결과 조회 API

query.getResultList()
- 결과가 하나 이상일 때, 리스트 반환
- 결과가 없으면 빈 리스트 반환

```java
List<Member> resultList = em.createQuery(...).getResultList();
```

query.getSingleResult()
- 결과가 정확히 하나일 때, 단일 객체 반환
- 결과가 없으면 `jakarta.persistence.NoResultException`
- 결과가 둘 이상이면 `jakarta.persistence.NonUniqueResultException`

```java
Member result = em.createQuery(...).getSingleResult();
```

### 파라미터 바인딩

이름 기준 바인딩

```java
Member result = em.createQuery("select m from Member m where m.username = :username")
		.setParameter("username", "member1")
		.getSingleResult();
```

위치 기준 바인딩

```java
Member result = em.createQuery("select m from Member m where m.username = ?1")
		.setParameter(1, "member1")
		.getSingleResult();
```

위치 기준 바인딩의 경우 위치가 바뀌게 되면 순서가 밀려 버그가 발생할 수도 있다. 

따라서 위치 기준 바인딩은 사용하지 말고, 위치가 바뀌어도 버그가 발생하지 않는 이름 기준 바인딩을 사용하는 것이 좋다. 