---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

자세한 내용으로 들어가기에 앞서 간단하게 어떤 어노테이션이 있는지 알아보자

- `@Column`: 컬럼 매핑
- `@Temporal`: 날짜 타입 매핑
- `@Enumurated`: enum 타입 매핑
- `@Lob`: BLOB, CLOB 타입 매핑
- `@Transient`: 특정 필드를 컬럼과 매핑하지 않음
- `@Access`: JPA가 엔티티에 접근하는 방식 설정


## @Column

엔티티의 필드를 테이블의 컬럼과 매핑한다.

### 속성

- name
	- 필드와 매핑할 테이블의 컬럼 이름
	- 기본 값은 객체의 필드 이름
- insertable/updatable
	- 등록/변경 가능 여부를 나타냄
	- `false`여도 데이터베이스에서 강제로 변경은 가능하지만 JPA를 쓸 때는 반영 안됨
	- 기본 값은 `true`
- nullable(DDL)
	- `null` 값의 허용 여부를 설정한다. `false`로 설정하면 DDL 생성 시에 `not null` 제약조건이 붙는다.
- unique(DDL)
	- `@Table`의 `uniqueConstraints`와 같지만 한 컬럼에 간단히 유니크 제약 조건을 걸 때 사용한다.
	- 하지만 컬럼에서 유니크 제약 조건을 거는 것은 그리 선호하지 않는 방법이다.
		- 제약조건의 이름이 무작위로 설정됨 -> `uniqueConstraints`는 이름 설정 가능
- columnDefinition(DDL)
	- 데이터베이스의 컬럼 정보를 직접 줄 수 있다. (자료형, 길이, 기본값 설정 가능)
		- `varchar(100) default 'EMPTY'`
	- 특정 데이터베이스의 종속적인 옵션들도 넣을 수 있다.
	- 기본 값은 필드의 자바 타입과 방언 정보를 사용
- length(DDL)
	- 문자 길이 제약조건, `String` 타입에만 사용한다.
	- 기본 값은 255
- precision/scale (DDL)
	- `BigDecimal` 타입에서 사용한다. (`BigInteger`도 사용할 수 있다)
	- 참고로 `double`, `float` 타입에는 적용되지 않는다.
	- `precision`은 소수점을 포함한 전체 자릿수를 정한다.
		- 기본 값은 19 (합쳐서 19개의 숫자)
	- `scale`은 소수의 자릿수를 정한다.
		- 기본 값은 2 (소수점 하위 2번째 자리)

## @Enumurated

`enum` 타입을 매핑할 때 사용한다.

### 속성

`@Enumurated`의 기본 값은 `EnumType.ORDINAL`이다.
- EnumType.ORDINAL: `enum`의 순서대로 데이터베이스에 저장
	- 선언된 순서를 데이터베이스에 반영하여 저장한다.
	- `USER, ADMIN`으로 선언했을 경우
		- 0: `USER`
		- 1: `ADMIN`
- EnumType.STRING: `enum`의 이름을 문자열로 데이터베이스에 저장
	- 선언된 이름을 데이터베이스에 반영하여 저장한다.

### 주의

`@Enumurated`를 사용 시 옵션으로 `ORDINAL`을 사용하지 않도록 한다. 

```java
public enum RoleType {  
    USER, ADMIN  
}
```

처음엔 위의 방식으로 `RoleType`을 선언하고 있다가 나중에 `GUEST`라는 `RoleType`이 새로 생겼다고 가정해보자.

```java
public enum RoleType {  
    GUEST, USER, ADMIN  
}
```

사람마다 다를 순 있겠지만 `GUEST`라는 값을 `ADMIN` 옆에 위치시키진 않을 것이다. 이렇게 새로운 `RoleType`을 선언한 뒤에 새로운 데이터가 들어온다면 기존에 데이터베이스에 있던 값은 어떻게 변할까?

간단하게 `id`, `name`, `role_type`을 컬럼으로 갖는 `Member` 테이블이 존재한다고 가정하고, 해당 테이블 안에 `USER`와 `ADMIN`을 `RoleType`으로 갖는 데이터가 각각 하나씩 존재한다고 했을 때 저장된 값은 다음과 같다.

```java
Member member1 = new Member();  
member1.setId(1L);  
member1.setName("A");  
member1.setRoleType(RoleType.USER);  
em.persist(member1);  
  
Member member2 = new Member();  
member2.setId(2L);  
member2.setName("B");  
member2.setRoleType(RoleType.ADMIN);  
em.persist(member2);  
  
tx.commit();
```

![ordinal_ex_1.png](/img/user/media/ordinal_ex_1.png) `GUEST` 선언 전

```java
Member member3 = new Member();  
member3.setId(3L);  
member3.setName("C");  
member3.setRoleType(RoleType.GUEST);  
em.persist(member3);  
  
tx.commit();
```

![ordinal_ex_2.png](/img/user/media/ordinal_ex_2.png) `GUEST` 선언 후

예시 코드와 결과를 보면 enum의 순서는 바뀌었지만 이미 저장된 데이터는 바뀌지 않아서 결국 어떤 역할을 가지고 있는지 알 수 없게 된다. 따라서 데이터 마이그레이션 등을 수행해야 하는 불상사가 생기게 된다.

비록 `ORDINAL`보다 데이터를 더 많이 차지하는 것은 `STRING`이지만 위험을 가지고 운영을 하는 것보단 `STRING` 방식으로 저장하는 것이 더 나은 선택이다.


## @Temporal

아래의 날짜 타입을 매핑할 때 사용한다.
- `java.util.Date`
- `java.util.Calendar`

> [!tip] 참고
> `LocalDate` 또는 `LocalDateTime`을 사용하는 경우 데이터베이스 저장 시 자동으로 `date`와 `timestamp`로 매핑되어 생성되기 때문에 굳이 `@Temporal` 어노테이션을 사용할 필요가 없다.

### 속성

- TemporalType.TIME: 시간을 데이터베이스 `time` 타입과 매핑
- TemporalType.TIMESTAMP: 날짜 + 시간을 데이터베이스 `timestamp` 타입과 매핑


## @Lob

데이터베이스의 `BLOB`, `CLOB` 타입과 매핑할 때 사용한다.

`@Lob`은 따로 지정할 수 있는 속성이 없다. 매핑하는 필드가 문자면 `CLOB`으로 매핑하고 그렇지 않다면 `BLOB`으로 매핑한다.

- BLOB(Binary Large OBject)
	- 이진 데이터를 저장할 때 사용되며, 주로 이미지, 오디오, 비디오, 바이너리 데이터 등을 저장할 때 사용된다.
	- 자바 자료형으로 분류
		- `byte[]`
		- `java.sql.BLOB` 
- CLOB(Character Large OBject)
	- 문자 기반의 큰 객체를 나타내며, 주로 텍스트 데이터를 저장할 떄 사용된다.
	- 자바 자료형으로 분류
		- `String`
		- `char[]`
		- `java.sql.CLOB`


## @Transient

엔티티의 필드를 매핑하고 싶지 않을 때 사용하는 어노테이션이다. 저장 및 조회도 하지 않기 때문에 메모리에서 사용하고 버려지는 값으로 사용할 수 있다.


## @Access

JPA가 엔티티 데이터에 접근하는 방식을 지정한다.

### 속성

- AccessType.FIELD
	- 필드에 직접 접근하며 `private`로 선언했더라도 접근이 가능하다.
	- `@Id`가 필드에서 선언되었을 경우 해당 접근 방식은 생략이 가능하다.
- AccessType.PROPERTY
	- 접근자(getter/setter)를 통해서 접근이 가능하다.
	- `@Id`가 프로퍼티에서 선언되었을 경우 해당 접근 방식은 생략 가능하다.