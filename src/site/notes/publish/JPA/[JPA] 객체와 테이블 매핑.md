---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

JPA에서 제일 중요하게 봐야 되는 두 가지 중 하나는 JPA가 내부적으로 어떤 매커니즘으로 동작하는지에 대한 **매커니즘적인 측면**과 객체와 관계형 데이터베이스를 어떻게 매핑을 해서 사용하는지에 대한 **정적인 측면**으로 나뉜다. 쉽게 말하면 **영속성 컨텍스트**와 **엔티티 매핑**의 두 가지로 나뉜다고 볼 수 있을 것 같다.

**엔티티 매핑**에는 다음과 같은 매핑 어노테이션이 있다. 
- 객체와 테이블 매핑
	- `@Entity`
	- `@Table`
- 필드와 컬럼 매핑
	- `@Column`
- 기본 키 매핑
	- `@Id`
- 연관관계 매핑
	- `@ManyToOne`
	- `@JoinColumn`

## @Entity

- `@Entity`가 붙은 클래스는 JPA가 관리하는 엔티티이다. 해당 어노테이션이 붙지 않으면 JPA와는 전혀 관계없는 그냥 내가 마음대로 쓰고 싶은 클래스라고 볼 수 있다.
- JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity`가 필수이다.
- 주의
	- 기본 생성자 필수 (파라미터가 없는 `public` 또는 `protected` 생성자)
	- `enum`, `interface`, `inner class`, `final class` 사용 불가
	- 저장할 필드에 `final` 사용 불가

### 속성

- name
	- JPA에서 사용할 엔티티의 이름을 지정한다. (JPA가 내부적으로 구분하는 이름)
	- 기본 값은 클래스의 이름을 그대로 사용하는 것이다.
	- 같은 클래스의 이름이 없으면 가급적 기본 값을 사용한다.


## @Table

- `@Table`은 엔티티와 매핑할 테이블을 지정한다.

### 속성

- name
	- 매핑할 테이블의 이름을 설정
	- 기본 값은 엔티티의 이름을 그대로 사용하는 것
- catalog
	- 데이터베이스 `catalog` 매핑
- schema
	- 데이터베이스 `schema` 매핑
- uniqueConstraints
	- DDL 생성 시 유니크 제약 조건 생성


## 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
	- 객체에서 매핑을 다 해놓으면 애플리케이션 사용할 때 필요하면 테이블을 다 만들어준다.
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 통해 데이터베이스에 맞는 적절한 DDL을 생성해 준다.
- 이렇게 생성된 DDL은 개발 장비에서만 사용해야 한다.
- 생성된 DDL은 운영 서버에서는 사용하지 않거나, 적절히 다듬은 후 사용한다.

### 속성

`persistence.xml` 설정 파일에 `hibernate.hbm2ddl.auto`의 값으로 아래의 속성을 적용한다.
- create
	- 애플리케이션 생성 시 기존 테이블 삭제 후 다시 생성한다.
	- 애플리케이션 실행 -> drop -> create -> 애플리케이션 종료
- create-drop
	- create와 같으나 종료 시점에 테이블을 drop 한다.
	- 애플리케이션 실행 -> drop -> create -> drop -> 애플리케이션 종료
- update
	- 기존 내용은 유지한 채 변경되는 것만 반영한다. (운영 DB에는 사용하면 안됨)
		- 필드 추가 -> `alter table ... add column ...`
	- 단, 필드 삭제의 경우는 반영하지 않는다.
		- 필드 삭제 -> `...` (반영 X)
- validate
	- 엔티티와 테이블이 정상 매핑되었는지만 확인한다.
		- 필드 추가 -> 실행 -> 에러 발생 (엔티티와 테이블의 매핑 정보가 다름)
- none (관례)
	- 사용하지 않는다.
	- `value = "asdjfakjhdsfklj"` 적는 것과 똑같지만 관례상 `none`으로 적는다. 
		- 매칭되는 것이 없기 때문에 실행이 안되는 것

### 주의

- 운영 장비에는 절대 `create`, `create-drop`, `update` 사용하면 안된다.
- 개발 초기에는 `create` 또는 `update`
- 테스트 서버는 `update` 또는 `validate`
- 스테이징과 운영 서버는 `none`

결국 이런 웹 애플리케이션에서 사용하는 DBMS 계정은 `alter`나 `drop`을 못하도록 계정을 분리하는 것이 맞다.


## DDL 생성 기능

데이터베이스 스키마 자동 생성과 다른 것이다. (내가 잘 몰랐던 내용이라 넣어봄)

- 제약 조건 추가: 회원 이름은 필수, 10자 제한
	- `@Column(nullable = false, length = 10)`
- 유니크 제약 조건 추가
	- `@Table(uniqueConstraints = {@UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"})})` 
	- `@Column(unique = true)`
- [DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다](https://www.inflearn.com/community/questions/847958/16-25%EC%B4%88-%EB%B6%80%ED%84%B0%EC%9D%98-%EC%84%A4%EB%AA%85%EC%9D%B4-%EC%9D%B4%ED%95%B4%EA%B0%80-%EA%B0%80%EC%A7%80-%EC%95%8A%EC%8A%B5%EB%8B%88%EB%8B%A4)
	- JPA의 런타임 시 기능은 결국 DB와 연결하여 `create`, `update`, `insert`, `delete` 쿼리를 날리는 것과 관련있다.
	- 이런 점에서 봤을 때, `@Table`의 `name` 속성을 바꾸면 JPA는 해당 `name`에 있는 테이블명에 `create`, `update`, `insert`, `delete` 쿼리를 날리기 때문에 런타임 기능에 영향을 준다고 한 것 같다.
	- 반면에 `@Column`의 속성값의 경우 DDL 생성이 켜져있을 때, 처음 애플리케이션 실행 시에만 DDL에서 작동할 뿐, JPA의 기능을 활용하는 런타임에서는 사용하지 않는다는 의미인 것 같다.

