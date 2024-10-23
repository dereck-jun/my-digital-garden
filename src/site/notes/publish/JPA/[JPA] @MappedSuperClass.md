---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-mapped-super-class/"}
---


## @MappedSuperClass

공통 매핑 정보가 필요할 때 사용한다. 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용한다.

아래 예시를 보면 공통으로 들어가는 정보인 `createdAt`와 `modifiedAt`을 `BaseEntity` 클래스에 모아서 사용하는 것을 볼 수 있다.

```java
@Getter
@MappedSuperClass
public abstract class BaseEntity {
	private LocalDateTime createdAt;
	private LocalDateTime modifiedAt;
}

@Entity
public class Post extends BaseEntity {
	...
}
```


`@MappedSuperClass`는 상속관계 매핑이 아니니 헷갈리지 말자.

`@MappedSuperClass`는 엔티티가 아니라 그저 속성만 내려주는 클래스이다. 그래서 해당 어노테이션이 적용되어 있는 클래스는 테이블과 매핑이 되지 않고, 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공한다.

같은 맥락으로 해당 어노테이션이 적용된 클래스의 타입으로는 조회와 검색이 불가능하다. 이것도 상속관계 매핑과 다른 점이라고 볼 수 있다. (상속관계 매핑은 가능)

해당 클래스는 직접 사용할 일이 없으므로 추상 클래스로 만드는 것을 권장한다.

> `@Entity` 클래스는 같은 `@Entity` 클래스나 `@MappedSuperClass`로 지정한 클래스만 상속 가능하다.