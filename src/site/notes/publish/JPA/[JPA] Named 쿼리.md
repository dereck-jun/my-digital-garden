---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-named/"}
---

## Named 쿼리란?

미리 정의해서 이름을 부여해두고 사용하는 JQPL이다. **어노테이션이나 xml에 정의**할 수 있으며 **정적 쿼리**이다. **애플리케이션 로딩 시점에 초기화 후 재사용**하며, **애플리케이션 로딩 시점에 쿼리를 검증**한다.


## 정의 방법

### 1. @NamedQuery 어노테이션을 사용하여 엔티티 클래스에 정의
```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query = "select m from Member m where m.username = :username"
)
public class Member {
	...
}
```

### 2. xml 파일에 정의
```xml
<!-- META-INF/persistence.xml -->
<persistence-unit name="demo">
	<mapping-file>META-INF/ormMember.xml</mapping-file>

<!-- META-INF/ormMember.xml -->
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
	<named-query name="Member.findByUsername">
		<query>
			<![CDATA[
				select m 
				from Member m 
				where m.username = :username
			]]>
		</query>
	</named-query>
	...
</entity-mappings>
```

> 환경에 따른 설정
> 1. xml이 항상 우선권을 가진다.
> 2. 애플리케이션 운영 환경에 따라 다른 xml을 배포할 수 있다.
> 	- 솔루션을 개발했는데 특정 상황마다 다르게 나가야 한다.
> 	- mapping 파일을 따로 배포하면 된다.

### 3. Spring Data JPA에서는 @Query 어노테이션을 사용하여 Repository 메서드에 직접 정의할 수 있다.
```java
public interface UserRepository extends JpaRepository<User, Long> {
	@Query("select m from Member m where m.username = :username")
	Optional<Member> findByUsername(String username);
}
```


## 장점 및 주의사항

### 장점

- 쿼리 문자열을 Java 코드와 분리하여 코드 구조화 개선
- 애플리케이션 시작 시 쿼리 파싱 및 검증으로 오류 조기 발견
- SQL 인젝션 방지를 위한 파라미터 바인딩 강제
- 쿼리 재사용성 향상 
- 성능 최적화 -> 애플리케이션 로딩 시점에 초기화하기 때문

### 주의사항

- 복잡한 동적 쿼리에는 적합하지 않음
- 엔티티에 많은 쿼리가 정의될 경우 클래스가 복잡해질 수 있음