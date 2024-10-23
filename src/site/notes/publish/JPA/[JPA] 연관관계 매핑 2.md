---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-2/"}
---

## 연관관계 정의 규칙

연관관계를 매핑할 때, 생각해야 할 것은 크게 3가지가 있다.

1. 방향
	- 단방향
	- 양방향
2. 연관관계의 주인
	- 연관관계에서 관리 주체
3. 다중성
	- 일대일(1:1)
	- 일대다(1:N)
	- 다대일(N:1)
	- 다대다(N:M)

>  연관관계 매핑 2에서는 다중성에 대한 내용이 있음


## 다중성

데이터베이스를 기준으로 다중성을 결정한다. 이때 연관관계는 대칭성을 가진다.
- 일대다(1:N) <--> 다대일(N:1)
- 일대일(1:1) <--> 일대일(1:1)
- 다대다(N:M) <--> 다대다(M:N)

이때 일대다 관계과 다대일 관계는 똑같은 내용이 아니라 각각 일을 주인으로 두었을 때의 관점과 다를 주인으로 두었을 때의 관점에서 봤을 때의 내용이다.

### 다대일(N:1)

다대일 관계는 다쪽에서 외래 키를 관리하는 형태이고, 가장 많이 사용하는 연관관계이다. 반대는 일대다 관계이다. 

회원(Member)와 팀(Team)으로 다대일 관계를 만들어 보자.
- 요구사항
	- 한 팀에는 여러 회원이 들어갈 수 있다.
	- 한 회원은 한 개의 팀에만 들어갈 수 있다.

**단방향 다대일 예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@ManyToOne
	@JoinColumn(name = "team_id")
	private Team team;
	
	...
}

@Entity
public class Team {
	@Id @GeneratedValue
	@Column(name = "team_id")
	private Long id;
	private String teamName;

	...
}
```

다대일이기 때문에 `Member` 클래스 쪽에 `@ManyToOne`을 사용해서 해당 클래스가 다(Many)라는 것을 지정하면서 `@JoinColumn(name = "team_id")`로 `Team` 클래스와 어떤 컬럼을 기준으로 조인할 것인지를 정해줬다.

**양방향 다대일 예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@ManyToOne
	@JoinColumn(name = "team_id")
	private Team team;
	
	...
}

@Entity
public class Team {
	@Id @GeneratedValue
	@Column(name = "team_id")
	private Long id;
	private String teamName;

	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<>();

	...
}
```

양방향의 경우 일(ToOne)에 해당하는 `Team` 클래스에 `@OneToMany`를 추가하고 연관관계의 주인을 `mappedBy`로 지정해준다. 이때 주인 객체에서 사용하는 변수명을 지정해주면 된다. 

이번 예제 코드의 경우 `Team` 클래스의 주인인 `Member` 클래스가 `Team`에 대한 변수로 `team`을 사용하고 있기 때문에 `mappedBy = "team"`으로 지정해줬다.


### 일대다(1:N)

여기서는 앞서 말한대로 일(One)쪽이 연관관계의 주인일 때를 말한 것이다. 즉, 일에서 외래 키를 관리하겠다는 말이다.

객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조이다.

추가로 `@JoinColumn`을 꼭 사용해야 한다. 그렇지 않으면 `@JoinTable` 방식을 사용하게 된다.
- 자동으로 중간 테이블을 하나 추가하게 됨 (기본 값임)

표준 스펙에서 지원은 하지만 권장하지는 않는다.

#### 일대다 단방향

**예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;
	
	...
}

@Entity
public class Team {
	@Id @GeneratedValue
	@Column(name = "team_id")
	private Long id;
	private String teamName;

	@OneToMany
	@JoinColumn(name = "team_id") // DB 관점에서 조인할 컬럼
	private List<Member> members = new ArrayList<>();

	...
}
```

예제 코드를 기준으로 일대다 관계는 `Team` 클래스의 `members` 값을 바꿨을 때 다른 테이블에 있는 외래 키를 `update` 해줘야 한다.

> 아까 일에서 외래 키를 관리한다고 했잖아요

위에서 말한 외래 키의 관리는 JPA가 조회, 저장, 수정, 삭제 등을 하는 작업에서의 주체를 말한 것이고, 여기서 말한 다른 테이블에 있는 외래 키는 데이터베이스의 관점에서의 외래 키를 말한 것이다. 

코드에서 `@OneToMany`를 사용한다고 그게 데이터베이스까지 영향을 미치는 것은 아니다. 해당 어노테이션은 JPA가 어떤 객체를 중심으로 관리할 것인가를 나타낸 것이다. 

실제 사용을 아래와 같이 할 수 있다.

```java
Member member = new Member();  
member.setName("memberA");  
  
em.persist(member);  // 실행 1
  
Team team = new Team();  
team.setName("teamA");  
team.getMembers().add(member);  
  
em.persist(team);  // 실행 2
```

`실행 1`에서는 별 문제 없이 `insert` 쿼리가 나가는 것을 알 수 있다. 

하지만 `실행 2`가 문제다.

`실행 2`에선 `team`을 `insert`하고 `member`를 `update`하는 쿼리가 나간다. 이때 `Member`에 실제 외래 키가 존재하기 때문에 `Team`에서 `Member`에 외래 키를 수정하려면 조인 후 `update` 쿼리를 날려야만 수정할 수 있는 문제가 생긴 것이다.
- `Team`을 수정했는데 `Member`가 수정이 되는 기적을 보게 됨
- `insert` 쿼리 실행 후 `update` 쿼리를 실행하게 되는 낭비가 발생
	- 성능상 이슈는 크게 없으나 낭비가 발생하는 것은 팩트

일대다 단방향이 필요한 경우라도 유지보수의 측면에서 바라볼 때 다대일 관계로 매핑하는 것이 더 수월하기에 다대일 방식을 추천한다.

#### 일대다 단방향 관계 정리

**일대다 단방향 매핑의 단점**
- 엔티티가 관리하는 외래 키가 다른 테이블에 있다.
	- 그냥 이것 하나만으로도 어마어마한 단점
- 연관관계 관리를 위해 추가적인 `update` 쿼리가 실행된다.

결론적으로 일대다 단방향 매핑보다는 객체적으로 설계가 덜 깔끔해지는 손해를 보더라도 다대일 양방향 매핑을 사용하는 것이 더 좋다. (그냥 일대다를 쓰지 말자)

> 일대다 양방향 매핑은 없나요?

일대다 양방향 매핑이라는 것은 공식적인 스펙으로 존재하진 않는다. 하지만 야매로 된다.

위의 일대다 단방향 예제 코드에서 `Member` 클래스의 내용만 추가했다.

**일대다 양방향 예제 코드**
```java
@Entity
public class Member {
	...
	
	@ManyToOne
	@JoinColumn(name = "team_id", insertable = false, updatable = false)
	private Team team;
	
	...
}

```

코드를 보면 일대다의 대칭이기 때문에 `@ManyToOne`을 추가했다. 

그 다음이 중요한데 `@JoinColumn`으로 조인할 테이블의 `id`를 설정해준 뒤에 `insertable`, `updatable` 속성을 `false`로 만들어서 매핑은 되어있고 값도 쓰는데, 최종적으로 `insert/update`를 하지 않는 것이다.

정말 필요할 때가 아주 가끔 있겠지만 양방향으로 객체 참조가 필요한 경우 그냥 **다대일 양방향 매핑을 사용**하도록 하자.



### 일대일(1:1)

일대일 관계는 그 반대도 일대일이다. 그래서 주 테이블이나 대상 테이블 중 어느 곳이든지 외래 키를 설정할 수 있다. 

주 테이블이 `Member`라고 가정했을 때 외래 키를 `Member`에 넣어도 되고, 주 테이블은 아니지만 `Team`에 외래 키를 넣어도 되는 것이다. 

추가로 외래 키에 데이터베이스 유니크 제약 조건을 추가해야 일대일 관계가 성립한다. 사실 굳이 제약 조건을 넣지 않아도 할 수는 있지만 그러면 애플리케이션 관리를 매우 잘해야 한다.

데이터베이스 입장에서는 외래 키에 데이터베이스 유니크 제약 조건이 추가가 된게 일대일 관계가 된다.

#### 일대일 주 테이블 단방향

**일대일 주 테이블 단방향 예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@OneToOne
	@JoinColumn(name = "locker_id")
	private Locker locker;
}

@Entity
public class Locker {
	@Id @GeneratedValue
	@Column(name = "locker_id")
	private Long id;
	private int number;
}
```

코드를 보면 알겠지만 다대일 단방향 매핑과 어노테이션만 다르지 거의 똑같다는 것을 알 수 있다.


#### 일대일 주 테이블 양방향

**일대일 주 테이블 양방향 예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@OneToOne
	@JoinColumn(name = "locker_id")
	private Locker locker;
}

@Entity
public class Locker {
	@Id @GeneratedValue
	@Column(name = "locker_id")
	private Long id;
	private int number;

	@OneToOne(mappedBy = "locker")
	private Member member;
}
```

마찬가지로 다대일 양방향 매핑처럼 외래 키가 있는 곳이 주인이기 때문에 반대쪽에 `mappedBy`를 적용해서 읽기 전용으로 만들어 준다.


#### 일대일 대상 테이블 단방향

불가능하다. 지원도 안되고 방법이 없다.


#### 일대일 대상 테이블 양방향

이 경우는 논란의 여지가 있다. 어떤 테이블에서 외래 키를 관리하는게 좋을 것인가를 생각해봐야 한다.

테이블의 경우 한 번 생성되면 변경이 매우 어렵다. 하지만 야속하게도 비즈니스 로직은 언제든 바뀔 수 있다. 

만약 `Member`가 여러 개의 `Locker`를 가질 수 있게 변경되었다면 `Locker`에 외래 키가 있는 것이 변경에 유연하다.

하지만 비즈니스에서 `Member`는 웬만하면 조회를 해와야 한다. 이런 입장에서 바라보았을 땐 별 다른 조인 없이 `member.getLocker()`로 `locker`에 대한 값을 확인할 수 있다는 것이 장점이 된다.

결론적으로 종합적으로 판단하고 결정해야 한다는 것이고, 코드 상으로 바라보았을 땐 일대일 주 테이블 양방향을 할 때와 똑같이 하면 된다.


#### 일대일 정리

- 주 테이블에 외래 키
	- 주 객체가 대상 객체의 참조를 가지는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 찾는 방식
		- 장점: 주 테이블만 조회해도 대상 테이블에 데이터까지 확인 가능
		- 단점: 값이 없으면 외래 키에 `null` 허용
	- 객체 지향 개발자 선호
	- JPA 매핑 편리
- 대상 테이블에 외래 키
	- 대상 테이블에 외래 키 존재
		- 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경했을 때 테이블 구조 유지 가능
		- 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨
			- `Member`를 로딩할 때 무조건 `Locker`를 뒤져서 확인해야 한다.
			- 어차피 쿼리가 나가게 됨 -> 값의 존재 여부를 알기 때문에 프록시를 만들 이유가 없음
			- 따라서 지연 로딩으로 설정해도 대상 테이블에 외래 키가 있으면 무조건 즉시 로딩이 된다. (지연 로딩 설정해도 쿼리만 한 번 더 나감)



### 다대다(N:M)

관계형 데이터데이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없기 때문에 사용하면 안된다.

대신 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

여기서 문제는 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계를 풀어낼 수 있다. 따라서 ORM은 중간 테이블을 임의로 만들어서 매핑을 해준다.

설명만 들으면 "자동으로 매핑을 해준다니 이거 완전 좋은거 아닌가요?" 라고 생각할 수 있기 때문에 왜 안되는지 알아보자

#### 다대다 단/양방향

**다대다 단/양방향 예제 코드**
```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@ManyToMany
	@JoinTable(name = "member_product")
	private List<Product> products = new ArrayList<>();	
}

@Entity
public class Product {
	@Id @GeneratedValue
	@Column(name = "product_id")
	private Long id;
	private String name;

	// 양방향의 경우 추가
	// @ManyToMany(mappedBy = "products")
	// private List<Member> members = new ArrayList<>();
}
```

예제 코드에선 `@JoinTable`을 사용해서 생성되는 테이블의 이름을 지정해줬지만 해당 어노테이션을 빼도 중간 테이블을 임의로 생성해준다. 

그 외 나머지는 다른 방법들과 동일하다.

그렇다면 왜 실무에서 사용하면 안될까?

#### 다대다 매핑의 한계

다대다 관계의 경우 데이터베이스로는 풀어낼 수 없는 관계이기 때문에 중간(연결) 테이블을 임의로 생성한다고 설명했었다. 

하지만 중간 테이블은 단순히 연결만 하고 끝나지 않는다. 중간 테이블에는 **추가적인 비즈니스 로직이 들어가는데 이런 로직을 추가할 수가 없다.** 또한 중간 테이블이 숨겨져 있기 때문에 **나도 모르는 복잡한 조인 쿼리가 발생**하게 된다.

위와 같은 이유로 실무에서 사용하면 안되는 것이다.

#### 다대다 한계 극복

그럼 이런 문제를 어떻게 해결할 수 있을까?

바로 `@ManyToMany`를 `@OneToMany` + `@ManyToOne`으로 바꾸고, 중간 테이블을 엔티티로 승격하는 것이다.

그렇다면 이런 형태의 매핑이 이뤄질 것이다.

```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String name;

	@OneToMany(mappedBy = member)
	private List<MemberProduct> memberProducts = new ArrayList<>();	
}

@Entity
public class MemberProduct {
	@Id @GeneratedValue
	@Column(name = "member_product_id")
	private Long id;

	@ManyToOne
	@JoinColumn(name = "member_id")
	private Member member;

	@ManyToOne
	@JoinColumn(name = "product_id")
	private Product product;
}

@Entity
public class Product {
	@Id @GeneratedValue
	@Column(name = "product_id")
	private Long id;
	private String name;

	@OneToMany(mappedBy = "product")
	private List<MemberProduct> memberProducts = new ArrayList<>();
}
```

이렇게 되면 중간 테이블에 내가 원하는 비즈니스 로직들을 추가해서 구현할 수 있게 된다.

번외로 `MemberProduct`의 PK를 지금처럼 따로 만들어 주는 것이 아닌 `member_id`와 `product_id`를 묶어서 PK로 설정하고 각각 FK로 쓰는 방법도 존재한다. 

어떤 방식을 사용할 것인지는 고민이 필요하지만 따로 `id`를 두고, 필요에 따라 제약 조건을 추가하는 것이 운영적 측면에서 더 도움이 될 수도 있다.

결론은 다대다는 사용하지 말자.