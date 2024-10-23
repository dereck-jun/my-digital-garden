---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-jpql/"}
---

## JPQL 타입 표현

- 문자: 작은 따옴표 안에 넣어주면 되고, 만약 작은 따옴표를 표현해야 되면 작은 따옴표 2개 넣으면 된다. 
	- 'HELLO'
	- 'She''s'
- 숫자: 자바와 거의 똑같이 사용하면 된다.
	- Long: 10L
	- Double: 10D
	- Float: 10F
- Boolean
	- TRUE
	- FALSE
- ENUM: 자바 패키지명을 전부 포함해서 넣어야 한다.
	- `com.example.domain.MemberType.ADMIN`
	- 파라미터 바인딩 사용하면 줄일 수 있음
- 엔티티 타입: 상속 관계에서 사용한다. (DTYPE에 대한 내용임)
	- `em.createQuery("select i from Item i where type(i) = Book", Item.class);`


## JPQL 기타

SQL과 문법이 같은 식이다.

- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL