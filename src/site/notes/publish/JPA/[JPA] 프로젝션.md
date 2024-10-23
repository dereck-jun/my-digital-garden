---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 프로젝션 - 조회

프로젝션은 SELECT 절에 조회할 대상을 지정하는 것이다.

프로젝션의 대상에는 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자 등 기본 데이터 타입)이 있다.

```sql
select m from Member m                      # 엔티티 프로젝션
select m.team from Member m                 # 엔티티 프로젝션
select m.addresss from Member m             # 임베디드 타입 프로젝션
select m.username, m.age from Member m      # 스칼라 타입 프로젝션
```

또한 DISTINCT로 중복 제거도 가능하다.

```sql
select distinct m.age from Member m where m.age > 20    # 중복 제거
```


### `em.createQuery()`로 가져온 결과들은 영속성 컨텍스트에 관리가 될까?

다음 예제 코드에서 `findMember.setAge(20)`가 정상적으로 동작해서 값이 변하면 영속성 컨텍스트에 관리가 된다는 의미이다.

```java
Member member = new Member();
member.setUsername("user");
member.setAge(10);
em.persist(member);

em.flush();
em.clear();

List<Member> members = em.createQuery("select m from Member m", Member.class)
		.getResultList();

Member findMember = members.get(0);
findMember.setAge(20);    // <-- 값이 변하면 관리가 되고 있는 것

tx.commit();
```

```
Hibernate: 
    /* update
        for hellojpa.jpql.Member */
        update Member 
    set
        age=?,
        team_id=?,
        username=? 
    where
        id=?
```

`update`가 실행되는 것으로 보아 관리가 된다는 것을 알 수 있다. 

결론은 `em.createQuery()`로 가져온 모든 값은 영속성 컨텍스트에 관리된다는 것을 알 수 있다.


### 다른 테이블에 있는 정보 불러오기

아래 예제 코드를 보면 `Member` 엔티티에서 `Member`와 관련된 `Team`을 조회할 수 있다.

```java
List<Team> teams = em.createQuery("select m.team from Member m", Team.class)  
        .getResultList();
```

```
Hibernate: 
    /* select
        m.team 
    from
        Member m */ select
            t1_0.id,
            t1_0.name 
        from
            Member m1_0 
        join
            Team t1_0 
                on t1_0.id=m1_0.team_id
```

그런데 하나 이상한 점이 있다. 

JPQL의 모습과는 다르게 SQL에서는 `inner join`을 통해 `Member`와 `Team`을 조인하는 쿼리가 나간다.

JPQL은 SQL과 비슷하게 써야 한다. 

조인 자체가 성능에 영향을 줄 수 있는 요소가 너무 많기도 하고, 튜닝할 수 있는 요소도 많기 때문에 어떤 쿼리가 나갈 것인지 예측할 수 있어야 하고, 그게 한눈에 보여야 한다.

즉, 조인은 웬만하면 명시적으로 해야 한다는 것이다.

따라서 이후 경로 표현식에서 나오겠지만, 위 코드는 다음과 같이 변경하는 것이 좋다.

```java
List<Team> teams = em.createQuery("select t from Member m join m.team t", Team.class)  
        .getResultList();
```

실행 결과는 동일하다.


### 임베디드 타입 조회하기

임베디드 타입 프로젝션의 경우 조회할 때 임베디드 타입으로 조회할 수는 없고, 그 값의 엔티티로부터 시작해야 한다.

해당 값이 어딘가에 소속이 되어있기 때문에 소속되어 있는 엔티티를 정해줘야 하는 것이다.

```java
em.createQuery("select o.address from Order o", Address.class).getResultList();

// em.createQuery("select a from Address a", Address.class).getResultList();
```


## 프로젝션 - 여러 값 조회

스칼라 타입 프로젝션의 경우 값의 반환 타입이 여러 개가 나올 수 있다.

```sql
select m.username, m.age from Member m
```

이때 사용할 수 있는 세가지 방법이 있다.
1. `Query` 타입으로 조회
2. `Object[]` 타입으로 조회
3. `new` 명령어로 조회
	- 단순 값을 DTO로 바로 조회
	- 패키지 명을 포함한 전체 클래스명 입력
	- 문자와 타입이 일치하는 생성자 필요


### Query 타입으로 조회

```java
Query query = em.createQuery("select m.username, m.age from Member m");  
Object result = query.getSingleResult();  // Object[2]@6189

Query query = em.createQuery("select m.username, m.age from Member m");  
List resultList = query.getResultList();  // resultList: size = 2
```

`result` 안에 `Object[]`이 있는 것을 알 수 있다. 해당 배열 안에는 순서대로 `m.username`의 값, `m.age`의 값이 있다.

만약 `getSingleResult()`가 아니라 `getResultList()`로 조회 후 반환했다면 `resultList` 안에 반환된 개수만큼의 `Object[]`이 존재하고, 각각의 배열 안에 값이 저장되어 있다.


### TypeQuery로 조회

```java
List<Object[]> resultList = em.createQuery(
		"select m.username, m.age from Member m").getResultList();
Object[] result = resultList.get(0);
```

제네릭에 `Object[]`를 선언하는 방법이다. Query 타입으로 했을 때와는 다르게 타입 캐스팅에 필요한 코드를 적을 필요가 없다는 특징이 있다.


### new 명령어로 조회

new 명령어로 조회를 하는 방법은 먼저 DTO 클래스를 생성한 뒤, 반환하려는 속성에 맞게 필드를 만들어주고, 문자와 타입에 맞게 생성자를 만들어주면 된다.

사용할 때는 `new` 뒤에 패키지명을 포함한 전체 클래스명을 적고, 괄호로 묶어서 찾으려는 속성을 정해주면 된다.

```java
class MemberDTO {  // 경로: hellojpa/jpql/MemberDTO.class
	private String username;  
	private int age;  
	  
	public MemberDTO(String username, int age) {  
	    this.username = username;  
	    this.age = age;  
	}
	// getter/setter
}

List<MemberDTO> result = em.createQuery(
		"select new hellojpa.jpql.MemberDTO(m.username, m.age) from Member m", 
		MemberDTO.class).getResultList();  
  
MemberDTO memberDTO = result.get(0);
```

이때 생성자에서 정의한 타입이나 순서를 바꿔서 사용하면 에러가 난다. 

아래는 생성자에서 정의한 타입/순서인 `String username, int age`를 `m.age, m.username`으로 바꾼 것이다. 

```java
class MemberDTO {  // 경로: hellojpa/jpql/MemberDTO.class
	private String username;  
	private int age;  
	  
	public MemberDTO(String username, int age) {  
	    this.username = username;  
	    this.age = age;  
	}
	// getter/setter
}

List<MemberDTO> result = em.createQuery(
		"select new hellojpa.jpql.MemberDTO(m.age, m.username) from Member m", 
		MemberDTO.class).getResultList();  
  
MemberDTO memberDTO = result.get(0);
```

직접 해보면 알겠지만 컴파일 에러가 나기 때문에 바로 알 수 있을 것이다.

> "생성자 'hellojpa.jpql.MemberDTO(int, java.lang.String)'를 해결할 수 없습니다"

