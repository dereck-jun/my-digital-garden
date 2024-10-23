---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## JPA의 데이터 타입 분류

엔티티 타입
- `@Entity`로 정의하는 객체
- 데이터가 변해도 식별자로 지속해서 추적 가능
	- ex) 회원 엔티티의 필드 값을 변경해도 식별자로 인식할 수 있다.

값 타입
- `int`, `Integer`, `String` 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
- 식별자가 없고 값만 있으므로 변경 시 추적 불가
	- ex) `int num = 10; -> num = 20` 처럼 값을 변경하면 완전히 다른 값으로 대체됨


## 값 타입 분류

- 기본 값 타입
	- 자바 기본 타입(`int`, `double`)
	- 래퍼 클래스(`Integer`, `Long`)
	- `String`
- 임베디드 타입(`embedded type`, 복합 값 타입)
- 컬렉션 값 타입(`collection value type`)


### 기본 값 타입

- 생명 주기를 엔티티에 의존한다
	- ex) 회원을 삭제하면 안에 있는 필드도 함께 삭제
- 값 타입은 공유하면 안됨
	- ex) 회원 이름 변경 시 다른 회원의 이름도 함께 변경되면 안됨 (나쁜 의미로서의 사이드 이펙트)

> 참고: 자바의 기본 타입은 절대 공유되지 않는다.
> - 기본 타입은 절대 공유되지 않는다.
> - 기본 타입은 항상 값을 복사한다. (참조 X)
> - 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체지만 변경은 안된다 (값을 변경할 방법이 없음)



### 임베디드 타입

임베디드 타입은 엔티티의 값이다. 따라서 크게 의미를 가질 필요가 없다. 

임베디드 타입 사용 전과 후에 **매핑하는 테이블은 같다.**

객체와 테이블을 아주 세밀하게 매핑하는 것이 가능하다.

> 참고: 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.

특징
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입이라 한다.
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함
- `int`, `String`과 같은 값 타입이다.
- 임베디드 타입의 값이 `null`이면 매핑한 컬럼의 값은 모두 `null`이다.

장점
- 재사용이 가능하다
- 클래스 내에서 응집도가 높다
- 해당 값 타입만 사용하는 의미 있는 메서드를 만들 수 있다
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명 주기에 의존한다

#### 사용법

- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
- 기본 생성자 필수

#### 임베디드 타입과 연관관계

엔티티는 임베디드 타입의 값을 가질 수 있고, 임베디드 타입의 값은 임베디드 타입의 값이나 엔티티를 가질 수 있다.

```
ENTITY -> VALUE -> VALUE   // 엔티티 -> 임베디드 -> 임베디드
ENTITY -> VALUE -> ENTITY  // 엔티티 -> 임베디드 -> 엔티티 
```

`@Column`을 사용해서 이름 변경 등도 가능

#### @AttributeOverride: 속성 재정의

한 엔티티에서 같은 값 타입을 사용하게 되면 컬럼명이 중복되는 문제가 생긴다.

이때 사용할 수 있는 어노테이션이 `@AttributeOverrides`, `@AttributeOverride`이다.

아래는 `Address` 클래스를 동시에 사용하고 있을 때 `@AttributeOverrides`로 속성 재정의를 하는 방법이다.

```java
@Entity
class Member {
	...
	@Embedded
	private Address homeAddress;

	@Embedded
	@AttributeOverrides({
		@AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
		@AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
		@AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
	})
	private Address workAddress;
}
```


### 값 타입 컬렉션

- 값 타입을 하나 이상 저장할 때 사용한다.
	- 셀렉트 박스처럼 단순하고, 추적할 필요도 없고, 값이 바뀌어도 `update` 할 필요가 없을 때 사용한다.
- `@ElementCollection`, `@CollectionTable` 어노테이션을 사용해서 매핑하면 된다.
- 데이터베이스 컬렉션을 같은 테이블에 저장할 수 없다.
	- 컬렉션의 경우 일대다 개념이기 때문에 DB 안에 한 테이블로 컬렉션을 넣을 수 있는 방법이 없다.
	- 따라서 별도의 테이블로 풀어내야 함
	- 즉, 컬렉션을 저장하기 위한 별도의 테이블이 필요하다는 뜻임
- 값 타입 컬렉션도 지연 로딩 전략을 사용한다.


> 참고: 값 타입 컬렉션은 영속성 전이(CASCADE) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.


#### 매핑 방법

```java
@Entity
class Member {
	...
	
	@ElementCollection  
	@CollectionTable(name = "FAVORITE_FOOD", joinColumns = 
		@JoinColumn(name = "MEMBER_ID")
	)  
	@Column(name = "FOOD_NAME")  
	private Set<String> favoriteFoods = new HashSet<>();  
	  
	@ElementCollection  
	@CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))  
	private List<Address> addressHistory = new ArrayList<>();
}
```

`@ElementCollection`으로 컬렉션 값 타입이라는 것을 명시하고, `@CollectionTable`로 자동으로 생성할 컬렉션 테이블의 이름과 어떤 식별자와 조인해서 사용할 것인지를 넣어주면 된다.

이때 `@Column`을 사용해서 테이블에 들어갈 컬럼의 이름을 변경할 수 있다.


#### 저장

```java
Member member = new Member();
member.setName("member1");
member.setHomeAddress(new Address("homeCity", "homeStreet", "10001"));

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("탕수육");

member.getAddressHistory().add(new Address("oldCity1", "st1", "10002"));
member.getAddressHistory().add(new Address("oldCity2", "st2", "10004"));

em.persist(member);
```


#### 조회

저장 예제 코드에서 이어지는 부분이라고 생각하고 보자.

```java
em.flush();
em.clear();

// 지연 로딩 전략으로 인해 Member만 조회됨
Member findMember = em.find(Member.class, member.getId());

// addressHistory 사용되는 시점에 로딩
List<Address> addressHistory = findMember.getAddressHistory();
for (Address address: addressHistory) {
	// 출력문
}

// favoriteFoods 사용되는 시점에 로딩
Set<String> favoriteFoods = findMember.getFavoriteFoods();
for (String favoriteFood : favoriteFoods) {
	// 출력문
}
```


#### 수정

저장 예제 코드에서 이어지는 부분이라고 생각하고 보자.

```java
em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());

// homeCity -> newCity
// findMember.getHomeAddress().setCity("newCity"); <- 사이드 이펙트로 큰일날 수 있음
findMember.setHomeAddress(new Address("newCity", "newStreet", "12345"));

// 치킨 -> 제육볶음
findMember.getFavoriteFoods().remove("치킨");  // equals 재정의 안되어 있으면 안됨
findMember.getFavoriteFoods().add("제육볶음");
```

다른 곳은 크게 어려움이 없는데 수정에서 알아둬야 하는 점이 조금 있다.

먼저 `homeAddress`를 수정하는 부분을 보자. 코드에 주석으로도 적혀 있지만 단순하게 `setter`로 값을 변경했다간 사이드 이펙트 문제가 생길 수도 있다. 

따라서 객체를 항상 불변 객체로 만들어 놓고, 값을 변경할 땐 아예 새로운 인스턴스를 만들어서 넣어줘야 한다. 만약 이전 코드를 재사용하고 싶다면 저장할 때 `Address`를 `setHomeAddress()` 안에서 사용하지 말고, 따로 빼서 객체를 만들어 주고 넣어주면 된다.

```java
Address address = new Address(...);

Member member = new Member();
member.setHomeAddress(address);

// 이후 변경 시
member.setHomeAddress(new Address("newCity", address.getStreet(), address.getZipcode()));
```

그 다음은 `favoriteFoods` 안에 있는 값을 제거하고 새로 추가하는 부분이다. 여기서 `remove()`의 인자로 보내는 "치킨"이라는 값을 찾으려면 내부적으로 `equals()`가 동작을 하게 된다.

이때 `equals()`를 재정의 하지 않았을 경우 "치킨"이라는 값과 동등한 값을 찾지 못해서 삭제가 되지 않게 되니 꼭 재정의 해주도록 하자.


#### 제약 사항

값 타입은 엔티티와 다르게 식별자 개념이 없다. 식별자라는 것이 생기면 그때부턴 값 타입이 아닌 엔티티가 되기 때문이다.

값은 변경되면 추적이 어렵다. 그래서 사이드 이펙트 관련 버그를 찾아내기 힘들다.

값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장해야 한다. 쉽게 말해 객체를 불변 객체로 만들어서 관리해야 한다는 뜻이다.

값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다. 따라서 `null`값이 있거나 중복으로 저장하면 안된다. (`null` 제약, `unique` 제약)


#### 값 타입 컬렉션 대안

실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려하는 것이 나을 수도 있다. 일대다 관계를 위한 엔티티를 만들고, 여기에 값 타입을 사용하는 것이다. 

> "엔티티로 wrapping 해서 값 타입을 엔티티로 승급시킨다" 

```java
@Entity
@Table(name = "ADDRESS")
class AddressEntity {
	@Id @GeneratedValue
	private Long id;

	private Address address;
}

@Entity
class Member {
	...
	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
	@JoinColumn(name = "MEMBER_ID")
	private List<AddressEntity> addressHistory = new ArrayList<>();
}
```

> 꼭 값을 변경하지 않는다고 해도 DB의 쿼리 자체를 다른 쪽에서 시작해서 가지고 와야 하거나 하는 경우는 전부 엔티티라고 보면 된다.
> - ex) 주소가 사라져도 이력이 남아야 한다. (엔티티)

## 값 타입의 비교

값 타입 비교를 위해선 먼저 동일성 비교와 동등성 비교의 차이를 알아야 한다.

- 동일성(identity) 비교: 인스턴스의 참조 값을 비교, =\= 를 사용한다.
- 동등성(equivalence) 비교: 인스턴스의 값을 비교, `equals()`를 사용한다.

값 타입은 동등성 비교를 해야하기 때문에 값 타입의 `equals()`를 적절하게 재정의해야 한다. 주로 모든 필드를 다 재정의해야 한다.

재정의를 하지 않고 `equals()` 사용 시 값이 같아도 `false`가 나온다.
- `equals()`의 기본은 =\= 비교이기 때문

```java
// equals() 재정의 전
Address address1 = new Address("city", "street", "10001");
Address address2 = new Address("city", "street", "10001");

System.out.println("address1 equals address2: ", (address1.equals(address2))); // false
```

재정의 시에는 자동으로 만들어 주는 대로 하는 것이 좋고, 때에 따라선 값의 비교를 필드 접근이 아닌 `getter`로 호출하거나 `getClass()` 대신 `instanceof`를 사용해야 될 수도 있다.

```java
@Override  
public boolean equals(Object o) {  
    if (this == o) return true;  
    if (o == null || getClass() != o.getClass()) return false;  
    Address address = (Address) o;  
    return Objects.equals(city, address.city) && Objects.equals(street, address.street) && Objects.equals(zipcode, address.zipcode);  
}

// 또는

@Override  
public boolean equals(Object o) {  
    if (this == o) return true;  
    if (o == null || getClass() != o.getClass()) return false;  
    Address address = (Address) o;  
    return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());  
}
```
