---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 다형성 쿼리란?

다형성 쿼리는 객체 지향 프로그래밍의 다형성 개념을 데이터베이스 쿼리에 적용한 것으로, 상속 구조를 가진 엔티티들에 대해 공통된 기준으로 조회할 수 있는 기능을 말한다.

다형성 쿼리는 상위 클래스 또는 인터페이스를 기준으로 하위 클래스들의 인스턴스를 함께 조회할 수 있는 기능을 제공한다. 이를 통해 다양한 하위 클래스의 객체를 하나의 쿼리로 가져올 수 있으며, 객체 지향 설계의 장점을 데이터베이스 설계에도 반영할 수 있다.

예를 들어 `Vehicle`이라는 상위 클래스와 이를 상속받는 `Car`, `Truck` 클래스가 있을 때, 다형성 쿼리를 사용하면 `Vehicle` 타입으로 모든 차량을 조회할 수 있다.

### TYPE

- 조회 대상을 특정 자식으로 한정
- ex) `Vehicle` 중에 `Car`, `Truck`을 조회해라
	- JPQL: `select v from Vehicle v where type(v) in (Car, Truck)`
	- SQL: `select v.* from vehicle v where v.DTYPE in ('car', 'truck')`


### TREAT (JPA 2.1)

- 자바의 타입 캐스팅과 유사
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
- from, where, select(하이버네이트 지원) 사용
- ex) 부모인 `Vehicle`과 자식 `Car`가 있다.
	- JPQL: `select v from Vehicle v where treat(v as Car).brand = 'BMW'`
	- SQL: `select v.* from Vehicle v where v.DTYPE = 'car' and v.brand = 'BMW'`