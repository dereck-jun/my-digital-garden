---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 경로 표현식이란?

.(점)을 찍어서 객체 그래프를 탐색하는 것.

```sql
select m.username
from Member m 
	join m.team.t
	join m.orders o
where t.name = 'teamA'
```

위의 예시에서 `m.username`의 경우 상태 필드로 객체 그래프를 탐색했다고 보면 되고, `m.team t`의 경우는 단일 값 연관 필드(ManyToOne, OneToOne)라고 하고, `m.orders o`의 경우는 컬렉션 값 연관 필드라고 한다.


## 경로 표현식 용어 정리

- 상태 필드(state field): 단순히 값을 저장하기 위한 필드

- 연관 필드(association field): 연관 관계를 위한 필드
	- 단일 값 연관 필드: @ManyToOne, @OneToOne -> 대상이 엔티티
	- 컬렉션 값 연관 필드: @OneToMany, @ManyToMany -> 대상이 컬렉션


## 경로 표현식 특징

- 상태 필드: 경로 탐색의 끝, 추가적인 탐색 불가
- 단일 값 연관 경로: 묵시적 내부 조인 발생, 이후 추가적인 탐색 가능
- 컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 추가적인 탐색 불가(단, size는 사용가능)
	- from 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능
	- ex) `select m From Team t join t.members m` -> 이 경우 select 프로젝션에서 `m.username` 등을 사용할 수 있다.


## 경로 탐색을 사용한 묵시적 조인 시 주의사항

- 묵시적 조인 시에는 항상 내부 조인이 사용된다.
- 컬렉션은 경로 탐색의 끝이다. 이후 탐색을 이어나가고 싶다면 명시적 조인을 통해 별칭을 얻어야 가능하다.
- 경로 탐색은 주로 select, where 절에서 사용하지만 묵시적 조인으로 인해 SQL의 from (join) 절에 영향을 준다.

> 실무 조언
> - 가급적 묵시적 조인 대신에 명시적 조인 사용
> - 조인은 SQL 튜닝에 중요 포인트
> - 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.

