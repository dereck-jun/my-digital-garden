---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 영속성 전이 - CASCADE

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다. 하나의 부모가 자식을 관리할 때 주로 사용한다.
- ex) 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장
	- 게시판에 첨부파일 경로 등 -> 한 게시판에서만 관리를 함

다음과 같은 경우에 사용한다.
1. 부모와 자식의 라이프 사이클이 거의 동일할 때
2. 단일 엔티티에 완전히 종속적일 때

연관관계 매핑 속성으로 `cascade`를 추가해서 설정할 수 있다.
- ex) `OneToMany(mappedBy = "...", cascade = CascadeType.ALL)`

CASCADE는 다음과 같은 속성이 있다.
- ALL: 모든 CASCADE를 적용
- PERSIST: 엔티티를 영속화할 때, 연관된 엔티티도 함께 유지
- REMOVE: 엔티티를 제거할 때, 연관된 엔티티도 모두 제거
- MERGE: 엔티티 상태를 병합할 때, 연관된 엔티티도 모두 병합
- REFRESH: 상위 엔티티를 새로고침할 때, 연관된 엔티티도 모두 새로고침
- DETACH: 부모 엔티티를 준영속화하면, 연관 엔티티도 준영속화 시킴

### 주의

영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다. 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다.

위에 설명한 것과 같이 소유자가 하나일 때만 사용해야 한다. 소유자가 둘 이상일 때 사용하면 의도하지 않은 방향으로 전이될 수 있음.
- ex) 부모 A, B에 자식 C가 있고, `A-C`, `B-C`로 `CASCADE` 되어 있을 때 `A` 삭제 시 `C`까지 삭제되어 의도치 않게 `B`에 연결되어있는 `C` 값이 `null`로 바뀌게 되는 참사 발생



## 고아 객체

부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하는 기능이다.

`orphanRemoval` 속성으로 자동 삭제 여부를 결정할 수 있다.

```java
@Entity
class Parent {
	@Id @GeneratedValue
	private Long id;
	private String name;

	@OneToMany(..., orphanRemoval = true)
	@JoinColumn(name = "parent")
	private List<Child> children = new ArrayList<>();
	...
}
```

`orphanRemoval` 속성이 켜져있을 경우 `List`에서 값이 사라지면 해당 자식 엔티티도 삭제된다.

```java
Child child1 = new Child();  
child1.setName("child1");  
  
Child child2 = new Child();  
child2.setName("child2");  
  
Parent parent = new Parent();  
parent.setName("parent1");  
parent.addChild(child1);  
parent.addChild(child2);  
  
em.persist(parent);  
em.flush();  
em.clear();  
  
List<Child> children1 = parent.getChildren(); 
  
for (Child child : children1) {  
    System.out.println("child = " + child.getName());  // 1
}  
  
Parent findParent = em.find(Parent.class, parent.getId());  
findParent.getChildren().remove(0);  
List<Child> children2 = findParent.getChildren();  
  
for (Child child : children2) {  
    System.out.println("child = " + child.getName());  // 2
}
```

이 경우 `1`번에선 2개의 항목이 나오지만 `2`번에선 `index = 0`에 해당하는 객체가 삭제돼서 1개의 항목만 나올 것이다.


### 주의

참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능이기 때문에 참조하는 곳이 하나일 때 사용하거나 특정 엔티티가 개인 소유할 때 사용한다.

`@OneToMany` 또는 `@OneToOne`만 사용 가능하다.

개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. 이것은 `CascadeType.REMOVE`처럼 동작한다. 



## 영속성 전이 + 고아 객체의 생명주기

영속성 전이와 고아 객체를 더하는 것은 `cascade = CascadeType.ALL` + `orphanRemoval = true`의 의미이다.

위의 둘을 전부 켜게 되면 다음과 같은 의미를 지닌다.
- 스스로 생명주기를 관리하는 엔티티는 `em.persist()`로 영속화, `em.remove()`로 제거한다. 
- 두 옵션을 전부 켰기 때문에 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있다.
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용하다.
	- 참고: [도메인 주도 설계 (3) - 애그리거트](https://velog.io/@gentledot/ddd-aggregate)

