---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

즉시 로딩과 지연 로딩의 경우 프록시에 대한 내용을 이해하고 있어야 한다.
- 참고: [[publish/JPA/[JPA] 프록시\|프록시]]

## 지연 로딩 (FetchType.LAZY)

지연 로딩의 경우 연관관계 매핑 어노테이션에 `fetch = FetchType.LAZY` 속성을 추가해서 선언해준다. 연관관계 어노테이션 중 `*ToMany`에 해당하는 어노테이션의 기본 값이다.

- ex) `@ManyToOne(fetch = FetchType.LAZY)`

지연 로딩을 사용하게 되면 객체를 DB가 아닌 프록시에서 가져온다. 프록시에서 객체를 가져올 경우 실제로 사용(초기화)되기 전까지 조회 쿼리를 사용하지 않는다.

예를 들어 `Member`와 `Team` 클래스가 있고, `Member`에 `@ManyToOne(fetch = FetchType.LAZY)`로 `Team`과 연관관계를 맺고 있다고 가정해보자.

```java
@Entity
class Member {
	@Id @GeneratedValue
	private Long id;
	private String name;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "team_id")
	private Team team;
	...
}
```

지연 로딩의 경우 `Member` 객체를 저장하고, `member`를 조회하게 되면 `member`는 DB에서 조회를 하고, 조회된 `member`안의 `Team` 객체는 프록시로 가져온다.

그리고 `Team`을 실제로 사용할 때 DB에서 조회 후 가져오게 된다.


`member` 객체를 조회할 때 굳이 `team` 객체를 조회할 필요가 없으므로 DB에 쿼리가 한 번만 나가게 돼서 성능상의 이점을 얻는다.


## 즉시 로딩 (FetchType.EAGER)

즉시 로딩의 경우 연관관계 매핑 어노테이션에 `fetch = FetchType.EAGER` 속성을 추가해서 선언해준다. 연관관계 어노테이션 중 `*ToOne`에 해당하는 어노테이션의 기본 값이다.

- ex) `@ManyToOne(fetch = FetchType.EAGER)`

즉시 로딩을 사용하면 무조건 조회하는 시점에 필요한 값들이 모두 들어가 있어야 한다.

예를 들어 `Member`와 `Team` 클래스가 있고, `Member`에 `@ManyToOne(fetch = FetchType.EAGER)`로 `Team`과 연관관계를 맺고 있다고 가정해보자.

```java
@Entity
class Member {
	@Id @GeneratedValue
	private Long id;
	private String name;

	@ManyToOne(fetch = FetchType.EAGER)
	@JoinColumn(name = "team_id")
	private Team team;
	...
}
```

즉시 로딩의 경우 `Member` 객체를 저장하고, `member`를 조회하게 되면 `Team` 객체의 사용 여부와 관계없이 조회 시점에 `team`이 무조건 들어가 있어야 하기 때문에 조회 시 두 엔티티를 조인해서 가져온다.


## 어떤 방법을 사용해야 할까?

비즈니스 로직에서 두 객체가 동시에 사용되는 빈도가 높다면 즉시 로딩을 사용할 것 같다. 하지만 그것은 어디까지나 이론에서 적용되는 것이고, 실제 실무에선 무조건 지연 로딩을 사용한다고 한다.

- 즉시 로딩 사용 시 예상치 못한 SQL 문제 발생
- 즉시 로딩은 JPQL에서 `N+1`의 문제를 발생시킴
	- 다음과 같은 JPQL을 사용했다고 가정
		- `em.createQuery("select m from Member m", Member.class).getResultList()`
	- 이때 실제 SQL은 이렇게 실행될 것이다.
		- `SQL: select * from Member`
	- 그럼 `Member` 객체를 전부 다 가져오게 되는데 이때 `Member` 내부를 보니 `Team`에 대한 매핑이 즉시 로딩으로 되어있는 것을 확인하고 SQL을 한 번 더 실행
		- `SQL: select * from Team where ...`
	- 그런데 이 쿼리가 `Member`의 개수만큼 나감
	- 그래서 `Member`의 개수 `N`에 최초 `Member`를 조회하는 쿼리까지 합한 `N+1`번 쿼리가 실행됨

결론은 모든 연관관계에 지연 로딩을 사용하고, JPQL 패치 조인이나, 엔티티 그래프 기능을 사용하자.

> JPQL 패치 조인과 엔티티 그래프는 후에 설명