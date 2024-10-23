---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 벌크 연산이란?

**벌크 연산(bulk operation)은** 우리가 일반적으로 잘 아는 **SQL의 update, delete 문**이다. **pk를 찍어서 단건을 update 하는 것을 제외한 나머지 모든 update나 delete 문**이라고 생각하면 된다.

한 번의 쿼리로 다수의 엔티티를 수정/삭제할 수 있어 성능상 이점이 있고, JPA는 update와 delete만을 지원하지만 하이버네이트의 경우 insert도 지원한다.
- ex) `insert into (select ...) ...`

여담으로 JPA는 벌크 연산보다는 실시간 단건 조회에 특화되어 있다.


## 사용 방법

### 1. JPQL을 사용

```java
int resultCount = em.createQuery("update Member m set m.age = 20")
		.executeUpdate();
```

JPQL 사용 시 `executeUpdate()` 메서드를 사용하여 실행한다. 

### 2. `@Modifying` 사용

```java
@Modifying
@Query("update Member m set m.firstname = :firstname where m.lastname = :lastname")
int setFixedFirstnameFor(
	@Param("firstname") String firstname, @Param("lastname") String lastname
);
```

> 반환 값은 벌크 연산으로 영향을 받은 row 수이다.


## 장점 및 주의사항

### 장점

- 대량 데이터 처리 시 성능 향상
- 네트워크 트래픽 감소
- 비즈니스 로직 간소화

### 주의사항

- 영속성 컨텍스트를 무시하고 DB에 직접 쿼리를 실행함
- 데이터 정합성에 문제가 생길 수도 있음
	- 영속성 컨텍스트와 DB 간 값의 차이가 존재


## 해결방안

1. 벌크 연산을 먼저 실행한 뒤 엔티티를 조회한다.
	- 1차 캐시에 값이 없기 때문에 영속성 컨텍스트를 초기화할 필요 없이 바로 조회해서 사용할 수 있음
2. 벌크 연산 후 영속성 컨텍스트를 초기화한다.
	- 1차 캐시에 있던 정보가 초기화 되면서, 값 조회 시 DB에서 찾아오게 됨
	- 만약 초기화하지 않고 사용한다면 DB에서 값을 가져와도 1차 캐시에 같은 pk 값이 있다면 DB에서 조회한 값을 쓰지 않고 1차 캐시에 있는 값을 사용하게 됨
3. `@Modifying(clearAutomatically = true)`를 사용하여 자동 초기화한다.
	- Spring Data JPA에서 제공함