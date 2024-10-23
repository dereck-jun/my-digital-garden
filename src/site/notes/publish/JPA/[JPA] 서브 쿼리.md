---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 서브 쿼리

쿼리 안에 또 쿼리를 만드는 것을 말한다. 특별한 경우를 제외하곤 DB에서 사용하는 것과 같다고 보면 된다.

#### 예제

- 나이가 평균보다 많은 회원
```java
String jpql = "select m from Member m where m.age > (select avg(m2.age) from Member m2)";
```

- 한 건이라도 주문한 고객
```java
String jpql = "select m from Member m where (select count(o) from Order o where m = o.member) > 0";
```

위의 예시를 보면 위에 있는 대상을 끌고 와서 사용하는 경우가 있고, 별개로 만들어서 사용하는 경우가 있는데 대상을 끌고 와서 사용하면 성능이 잘 안나온다. 


### 서브 쿼리 지원 함수

- `[NOT] EXISTS (subquery)`: 서브 쿼리에 결과가 존재하면 참
	- `{ALL | ANY | SOME} (subquery)`
		- ALL: 모두 만족하면 참
		- ANY, SOME: 같은 의미로 조건을 하나라도 만족하면 참
- `[NOT] IN (subquery)`: 서브 쿼리의 결과 중 하나라도 같은 것이 있으면 참


#### 예제

- 팀 A 소속인 회원
```java
String jpql = "select m from Member m where exists (select t from m.team t where t.name = 'teamA')";
```

- 전체 상품 각각의 재고보다 주문량이 많은 주문들
```java
String jpql = "select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)";
```

- 어떤 팀이든 팀에 소속된 회원
```java
String jpql = "select m from Member m where m.team = ANY (select t from Team t)";
```


### 한계

- JPA 표준 스펙 상 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능 (하이버네이트 지원)
- FROM 절의 서브 쿼리는 현재 JPQL에선 불가능함
	- FROM 절의 서브 쿼리는 대부분 조인으로 풀 수 있다. 
	- 따라서 조인으로 풀 수 있다면 풀어서 해결
	- 풀 수 없을 경우 네이티브 쿼리나 쿼리를 두 번 날리는 식 등으로 해결

