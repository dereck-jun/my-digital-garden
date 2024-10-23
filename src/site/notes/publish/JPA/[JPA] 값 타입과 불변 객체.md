---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

> 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 
> 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.


## 값 타입 공유 참조

임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다. 

```java
Address address = new Address("city", "street", "10001");    // city, street, zipcode 순

Member member1 = new Member();
member1.setName("member1");
member1.setAddress(address);
em.persist(member1);

Member member2 = new Member();
member2.setName("member2");
member2.setAddress(address);
em.persist(member2);

member1.getAddress().setCity("new-city");
```

위와 같이 `member1`, `member2`가 같은 `address`를 보고 있을 때 개발자는 `member1`의 값만 바꾸려는 의도로 코드를 작성했겠지만 위 코드를 실제로 실행시키면 `member1`과 `member2`의 값이 모두 바뀌어 있는 것을 확인할 수 있다.

이런 현상을 **부수 효과(side effect)**라고 하는데 이런 종류의 버그는 잡기 많이 힘들다.

값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하기 때문에 값(인스턴스)를 복사해서 사용한다.

위의 코드를 수정해보자

```java
Address address = new Address("city", "street", "10001");    // city, street, zipcode 순

Member member1 = new Member();
member1.setName("member1");
member1.setAddress(address);
em.persist(member1);

Address copyAddress = 
	new Address(address.getCity(), address.getStreet(), address.getZipcode());

Member member2 = new Member();
member2.setName("member2");
member2.setAddress(copyAddress);
em.persist(member2);

member1.getAddress().setCity("new-city");
```

기존 `address`의 값을 복사했기 때문에 의도한 대로 `member1`의 값만 바뀐다.


## 객체 타입의 한계

- 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 없다. (side effect)
- 여기서 문제는 **임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입이 아닌 객체 타입**이 된다.
- 기본 타입에 값을 대입하면 값을 복사하지만 **객체 타입은 참조 값을 직접 대입**하고, **이것을 막을 방법이 없다.**

> 결론: 객체의 공유 참조를 피할 수 없게 된다.



## 불변 객체

불변 객체란 생성 시점 이후 절대 값을 변경할 수 없는 객체를 말한다.

생성자로만 값을 설정하고 수정자를 만들지 않으면 손쉽게 불변 객체로 만들 수가 있다.

이런 방법을 통해 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단할 수 있다. 따라서 값 타입은 불변 객체(`immutable object`)로 설계해야 한다.

불변 객체로 바꾼 뒤에 값을 변경할 일이 생긴다면 해당 객체를 다시 생성하고 바꿔끼워줘야 한다.

> 참고: `Integer`, `String`은 자바가 제공하는 대표적인 불변 객체이다.

