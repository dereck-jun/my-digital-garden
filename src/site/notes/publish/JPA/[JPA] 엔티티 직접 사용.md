---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 기본 키 값

- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용
- JPQL
	- `select m from Member m where m.id = :memberId` -> 엔티티의 아이디를 사용
	- `select m from Member m where m = :member` -> 엔티티를 직접 사용
- SQL
	- `select m.* from Member m where m.id = ?` -> 두 JPQL 모두 동일하게 실행됨


## 외래 키 값

- 기본 키 값을 사용할 때와 같다.
- JPQL
	- `select m from Member m where m.team.id = :teamId` -> 외래키의 아이디를 사용
	- `select m from Member m where m.team = :team` -> 외래키의 엔티티를 직접 사용
- SQL
	- `select m.* from Member m where m.team.id = ?`


> 추가 정보
> 1. `select m.team from Member m` -> 묵시적 조인 발생 (team 엔티티의 값을 select 해야하기 때문에 묵시적 조인 발생)
> 2. `select m from Member m. where m.team = :team` -> 묵시적 조인 발생X (team 엔티티 자체가 외래키를 가르키기 때문에 묵시적 조인이 발생하지 않음)