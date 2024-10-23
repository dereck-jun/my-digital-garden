---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## JPQL 소개

JPA를 사용하면 **엔티티 객체를 중심으로 개발**하게 된다. 

이때 문제는 검색 쿼리인데 검색을 할 때도 **테이블이 아닌 엔티티 객체를 대상으로 검색**을 해야하는데 **모든 DB의 데이터를 객체로 변환해서 검색하는 것을 불가능**하다. 이것은 데이터를 다 퍼올려서 메모리에 올려놓고 돌린다는 것과 같은 말이기 때문이다.

그래서 결국 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 **검색 조건이 포함된 SQL이 필요**하다. 

이런 문제를 해결하기 위해 JPA는 SQL을 추상화한 **JPQL**이라는 객체 지향 쿼리 언어를 제공한다. 

JPQL은 **객체를 대상으로 검색하는 객체 지향 쿼리**이고, **SQL을 추상화 했기 때문에 특정 데이터베이스 SQL에 의존적이지 않다**. 한마디로 정의하면 **객체 지향 SQL**이라고 볼 수 있다.

이러한 JPQL은 **SQL과 문법이 유사**하고, **ANSI 표준 SQL**이 지원하는 문법(`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`)은 **전부 지원**한다.

```java
List<Member> result = em.createQuery(
	"select m from Member m where m.age > 20", Member.class).getResultList();
```


> 차이점
> - SQL: 데이터베이스 테이블을 대상으로 쿼리
> - JPQL: 엔티티 객체를 대상으로 쿼리


## Criteria 소개

Criteria는 문자가 아닌 자바 코드로 JPQL을 작성할 수 있게 한다. 

자바 코드로 JPQL의 빌더 역할을 하는 것이다. 그리고 JPA의 공식 기능(스펙)이다.

하지만 Criteria는 너무 복잡하고 실용성이 없다. 오죽하면 그냥 `String`으로 사용하는게 더 낫다고 느껴질 정도이다.

그래서 Criteria 대신 **QueryDSL** 사용을 권장한다.

```java
// Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// Root 클래스 설정 (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

// 쿼리 생성
CriteriaQuery<Member> cq = query.select(m);

// 동적 쿼리 구현
if (username != null) {
	cq = cq.where(cb.equal(m.get("username"), "kim"));
}

List<Member> resultList = em.createQuery(cq).getResultList();
```


## QueryDSL 소개

QueryDSL은 **문자가 아닌 자바 코드로 JPQL을 작성**할 수 있게 해준다.

Criteria와 마찬가지로 **JPQL의 빌더 역할**을 한다. 

**컴파일 시점에 문법 오류를 찾을 수 있다는 큰 장점**이 있고, **동적 쿼리 작성이 편리**하다는 장점이 있다.

하지만 초기 Q클래스 등 **사용을 위한 설정이 좀 복잡**하다.

그래도 사용 자체는 JPQL을 안다면 단순하고 명확하기 때문에 **실무 사용을 권장**한다.

> 참고: [QueryDSL Reference Guide Page](http://querydsl.com/static/querydsl/5.0.0/reference/html_single/)


## native SQL 소개

JPA가 제공하는 SQL을 직접 사용하는 기능이다.

JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용할 수 있다.
- ex) Oracle의 `CONNECT BY`, 특정 DB만 사용하는 SQL 힌트 등

```java
String sql = "select member_id, username, age, city from member";

List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```


## JDBC 직접 사용, SpringJdbcTemplate 등

JPA를 사용하면서 JDBC Connection을 직접 사용하거나, SpringJdbcTemplate, MyBatis 등을 함께 사용할 수 있다.

단, 영속성 컨텍스트를 적절한 시점에 강제로 플러시 해줘야 한다.
- ex) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트 수동 플러시

```java
Member member = new Member();
member.setUsername("member1");
em.persist(member);    // member가 영속성 컨텍스트 안에 머물러 있음

// DB Connection을 가져와서 직접 쿼리 작성
// connection.executeQuery("select * from member");
// em.persist(member)가 아직 커밋 또는 플러시 되지 않아서 결과가 안나옴 
// -> 수동 플러시 필요

tx.commit();
```