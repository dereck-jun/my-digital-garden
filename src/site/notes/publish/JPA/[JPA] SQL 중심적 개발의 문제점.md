---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-sql/"}
---

## SQL 중심적인 개발의 문제점

SQL 중심적인 개발에는 여러 문제점이 있다.

1. 무한 반복, 지루한 코드
2. 패러다임의 불일치
3. 객체와 RDB의 차이
4. 모델링에서의 문제
5. 객체 그래프의 탐색이 불가능
6. 객체 비교에서 차이

### 1. 무한 반복, 지루한 코드

- CRUD 쿼리 무한 반복. 객체에 필드가 추가되면 쿼리에 변경점을 하드 코딩해줘야 한다.

### 2. 패러다임의 불일치

- 객체 지향의 목표와 RDB의 패러다임에 차이 때문에 문제가 생긴다.
	- 객체 지향: 추상화, 캡슐화, 상속, 다형성 등 시스템의 복잡성을 제어할 수 있는 다양한 장치들을 제공
	- RDB: 데이터를 중심으로 구조화되어 있음. (다양한 도구가 존재하지 않음)
- RDB가 인식할 수 있는 것은 SQL. 결국 객체를 SQL로 짜야 한다.
	- 객체 -> SQL 매핑 작업 -> RDB 저장
	- 위의 매핑 작업을 할 수 있는 것은 개발자 뿐. `개발자 == SQL Mapper`라고 할 정도

### 3. 객체와 RDB의 차이

#### 상속

객체에는 상속 관계가 있지만 RDB에는 상속 관계가 없다. 대신 상속 관계와 유사한 모델인 슈퍼타입, 서브타입이 존재한다.

![rdb_sup_sub_type.png.png](/img/user/media/rdb_sup_sub_type.png.png)

`album`을 DB에 저장한다고 하면 `item` 테이블 삽입 sql, `album` 테이블 삽입 sql 총 2개를 작성해야 한다. album을 조죄한다고 해도 `join`으로 `album`과 `item` 테이블을 조회해서 나온 결과 값을 일일이 각 객체의 필드 값에 넣어줘야 한다.

이는 개발자가 실수할 가능성을 높이게 되고, 생산성의 저하로 이어지게 된다.

하지만 자바 컬렉션을 사용하듯이 사용하게 된다면 훨씬 편하게 사용할 수 있을 것이다.
```java
list.add(album);  // 저장

Album album = list.get(album.getId());  // 조회

Item item = list.get(album.getId());  // 다형성 활용
```

#### 연관 관계

`Member` 객체는 `Member.team` 필드에 `Team` 객체의 참조를 보관해서 `Team` 객체와 관계를 맺는다. 이 참조 필드에 접근해 `Member`와 연관된 `Team`을 조회할 수 있다. 

반면 `MEMBER` 테이블은 `MEMBER_ID` 외래 키 컬럼을 사용해서 `TEAM`과 관계를 맺는다.  이 외래키를 사용해 `TEAM` 테이블과 조인하면 `MEMBER` 테이블과 연관된 `TEAM` 테이블을 조회할 수 있다. 

여기서 발생하는 문제는 객체의 참조 방향이다. 객체 연관 관계의 경우 `member.getTeam`으로 참조 가능하지만 반대로 `Team.getMember()`는 불가능하다. 반면 테이블은 어느 쪽에서든 조인을 사용할 수 있다.


### 4. 모델링에서의 문제

객체는 참조를 통해 관계를 맺는다. 

```java
class Member {
	Long memberId;
	Team team;
	String name;

	...
}

class Team {
	Long teamId;
	String name;

	...
}
```

그런데 위 코드처럼 객체 모델을 사용하면 테이블에 저장하거나 조회하기 쉽지않다. 객체 모델을 `Team` 객체로 DB는 `Team`을 `team_id`로 저장하기 때문이다. 이런 차이가 있어 개발자가 중간에서 변환 역할을 해야 한다.

```java
Member member = new Member();

// 회원 관련 정보
...

Team team = new Team();

// 팀 관련 정보
...

member.setTeam(team);  // 회원과 팀 관계 설정

```

이런 과정은 모두 패러다임 불일치를 해결하기 위해 소모되는 비용이다.


### 5. 객체 그래프 탐색

객체에서 회원이 소속된 팀을 조회할 때 다음처럼 참조해서 사용하면 연괸된 팀을 찾을 수 있다. 이것을 **객체 그래프 탐색**이라고 한다.

![entity_graph.png](/img/user/media/entity_graph.png)

```java
Team team = member.getTeam();

member.getOrder().getOrderItem();
```

객체는 마음대로 객체 그래프를 탐색할 수 있어야 한다. 그런데 DB에서는 객체를 조회할 때 `Tember`와 `Team`의 데이터만 조회했다면 `member.getOrder()`의 값은 `null`이 된다. 

SQL을 직접 다루면 처음 실행하는 SQL문에 따라 객체 그래프의 탐색이 한정된다. 이는 개발자에겐 큰 제약이며, 비즈니스 로직에 따라 사용하는 객체 그래프가 다른데 언제 끊어질 지 모를 객체 그래프를 함부로 탐색할 순 없다.


### 6. 객체 비교에서 차이

DB는 기본 키의 값으로 각 행을 구분한다. 반면, 객체는 동일성 비교와 동등성 비교의 두 가지 방법이 있다. 

```java
public Member getMember(Long memberId) {
	String sql = "select * from Member where member_id = ?";
	...
	// JDBC API, SQL 실행
	return new Member(...);
}
```

```java
Long memberId = 100L;
Member member1 = member.getMember(memberId);
Member member2 = member.getMember(memberId);

member1 == member2  // false
```

같은 `sql`, 같은 `member_id`로 조회했는데도 두 객체가 **동일**하지 않다. 그 이유는 내용은 같지만 `new`를 사용해서 새로운 인스턴스로 만들었기 때문이다. 만약 객체를 컬렉션에 보관했다면 동일성 비교에 성공했을 것이다. 

이런 패러다임 불일치를 해결하기 위해 같은 로우를 조회할 때마다 같은 인스턴스를 반환하도록 구현하는 것은 쉽지 않다. 여기에 트랜잭션이 동시에 실행되는 상황까지 고려하면 문제는 더 어려워질 것이다.