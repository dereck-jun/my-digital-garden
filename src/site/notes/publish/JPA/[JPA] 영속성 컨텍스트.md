---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 영속성 컨텍스트란?

- JPA를 이해하는데 가장 중요한 용어
- "엔티티를 영구 저장하는 환경"이라는 뜻
- EntityManager.persist(entity);
- DB에 저장한다는 것이 아니라 영속성 컨텍스트를 통해서 엔티티를 영속성화 한다는 뜻
	- 엔티티를 영속성 컨텍스트라는 곳에 저장한다는 말
- EntityManager 안에 영속성 컨텍스트라는 눈에 보이지 않는 어떤 공간이 생긴다고 이해하면 됨


## EntityManagerFactory / EntityManager

### EntityManagerFactory 특징
> 1. 생성 비용이 크다
> 2. 생성 비용이 크기 때문에 데이터베이스당 하나만 만들어서 쓰레드간 공유한다.
> 3. Thread-safe 하다

### EntityManager 특징
> 1. persist(), find(), remove() 등을 사용해 Entity를 관리한다.
> 2. Thread-safe 하지 않다.
> 3. 생성 시 데이터베이스 연결이 꼭 필요한 시점까지 커넥션을 얻지 않는다.

## 엔티티의 생명 주기

- 비영속 (new/transient): 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속 (managed): 영속성 컨텍스트에 관리되는 상태
- 준영속 (detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 (removed): 삭제된 상태

### 비영속

JPA와 전혀 관계없는 상태일 때 비영속 상태라고 한다. 아래 코드는 객체를 생성만 했을 뿐 JPA와는 전혀 관련없다.
- 객체를 생성한 상태

```java
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId(100L);
member.setName("member1");
```


### 영속

객체를 생성한 후 `EntityManager.persist()`에 객체를 저장한 상태를 말한다.

```java
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId(100L);
member.setName("member1");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// 객체를 저장한 상태 (영속)
em.persist(member);
```

이때 `em.persist()`의 경우에만 영속이 되는 것이 아니라, `em.find()`를 했을 때 1차 캐시에 없는 데이터를 찾아서 1차 캐시에 해당 정보가 저장된 경우도 영속 상태가 된다.

### 준영속 / 삭제

객체를 영속성 컨텍스트에서 분리하거나 객체를 삭제한 상태
- 삭제의 경우
	- 커밋이 되지 않은 영속 객체를 삭제할 경우 준영속과 같은 상태가 된다. 
	- 이미 데이터베이스에 저장된 객체를 찾아서 삭제할 경우 해당 데이터가 삭제된다.

```java
// 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId(100L);
member.setName("member1");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

// 객체를 저장한 상태 (영속)
em.persist(member);

// 객체를 영속성 컨텍스트에서 분리한 상태 (준영속)
em.detach(member);

// 객체를 삭제한 상태 (삭제)
Member findMember = em.find(Member.class, 1L);
em.remove(findMember);
```


## 영속성 컨텍스트의 이점
### 1차 캐시

- 1차 캐시는 영속 상태의 엔티티가 실제로 저장되는 곳이다
- 메모리에 Map<ID, Entity>의 형태로 존재한다고 생각
- 객체 생성 후 영속화하면 1차 캐시에 저장된다.
- 이때 객체를 조회할 경우 데이터베이스를 조회하기 전 1차 캐시 먼저 조회를 한다.
- 1차 캐시에 없을 경우 데이터베이스에 조회 후 1차 캐시에 저장한 뒤 해당 객체를 반환한다.

> 결론: 1차 캐시를 통해 성능상 이점을 가진다.

### 동일성(identity) 보장

- 동일성(Identity): 실제 인스턴스의 참조 값이 같다. (=\=)
- 동등성(Equality): 실제 값이 같다. (equals)

> 결론: (같은 트랜잭션 내에서) 같은 식별자에 대한 객체 조회는 동일성을 보장해준다.

### 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)

- 여러 데이터를 저장한다는 가정 하에 `EntityManager`는 쓰기 지연 SQL 저장소에 `insert sql`을 쌓아둔다.
	- 이때 저장소의 크기는 `<property name="hibernate.jdbc.batch_size" value="10"/>`로 지정할 수 있다.
	- 지정한 `value` 마다 `insert query`가 실행된다.
	- `batch-size`에 대한 제한이 없으면 `OutOfMemoryException`이 발생할 수도 있고, 메모 관리 측면에서도 효율적이지 않다.
	- 자세한 사항은 [여기](https://cheese10yun.github.io/jpa-batch-insert/#jpa-with-batch-insert-code-1)를 참조
- 이후 커밋 시점에 비로소 모든 쿼리를 데이터베이스에 보내게 된다. (flush)
- (문제가 없다면) 커밋이 정상적으로 수행된다.

> 결론: 영속 시점에 쿼리를 바로 데이터베이스에 보내는 것이 아니라 커밋 직전까지 쓰기 지연 SQL 저장소에 모아둔 뒤 한번에 보내기 때문에 성능상 이점이 있다.  

### 변경 감지 (Dirty Checking)

- 객체 수정 시 별도의 메서드 없이 객체를 수정할 수 있는 이유이다.
	1. JPA는 객체를 영속성 컨텍스트에 보관 시 스냅샷을 만들게 된다. (백업 데이터 같은 느낌)
	2. `flush()` 시점에 스냅샷과 객체를 비교해서 변경된 객체를 찾는다. 이때 영속성 컨텍스트 안에 있는 객체만 변경 감지 기능의 대상이 된다.
	3. 변경된 객체가 존재할 경우, 수정 쿼리를 쓰기 지연 SQL 저장소에 저장한다.
	4. 커밋 시점에 저장소 안에 있는 `sql`을 데이터베이스에 보낸다. (`flush()` 끝)
	5. (문제가 없을 경우) 트랜잭션이 정상적으로 종료(커밋)된다.

### 지연 로딩 (Lazy Loading)

[[publish/JPA/[JPA] 즉시 로딩과 지연 로딩\|[JPA] 즉시 로딩과 지연 로딩]]을 참고