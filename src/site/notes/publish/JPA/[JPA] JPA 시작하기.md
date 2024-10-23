---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-jpa/"}
---

## 설정
- java - maven 사용
- java 17
- pom 4.0.0
### 종속성
```xml
<!-- JPA 하이버네이트 -->  
<dependency>  
    <groupId>org.hibernate</groupId>  
    <artifactId>hibernate-core</artifactId>  
    <version>6.4.2.Final</version>  
</dependency>  
  
<dependency>  
    <groupId>javax.xml.bind</groupId>  
    <artifactId>jaxb-api</artifactId>  
    <version>2.3.1</version>  
</dependency>  
  
<!-- H2 데이터베이스 -->  
<dependency>  
    <groupId>com.h2database</groupId>  
    <artifactId>h2</artifactId>  
    <version>2.2.224</version>  
</dependency>
```

### persistence.xml

`resource/META-INF` 하위에 `persistence.xml` 생성 후 아래와 같이 작성
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<persistence version="2.2" xmlns="http://xmlns.jcp.org/xml/ns/persistence"  
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">  
    <persistence-unit name="hello">  
        <properties>  
            <!-- 필수 속성 -->  
            <property name="jakarta.persistence.jdbc.driver" value="org.h2.Driver"/>  
            <property name="jakarta.persistence.jdbc.user" value="sa"/>  
            <property name="jakarta.persistence.jdbc.password" value=""/>  
            <property name="jakarta.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>  
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>  
  
            <!-- 옵션 -->  
            <property name="hibernate.show_sql" value="true"/>  
            <property name="hibernate.format_sql" value="true"/>  
            <property name="hibernate.use_sql_comments"  value="true"/>  
<!--            <property name="hibernate.hbm2ddl.auto" value="create" />-->  
        </properties>  
    </persistence-unit>  
</persistence>
```


## 예제 코드

```java
public class JpaMain {  
  
    public static void main(String[] args) {  
  
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");  
        EntityManager em = emf.createEntityManager();  
        EntityTransaction tx = em.getTransaction();  
  
        tx.begin();  
  
        try {  
            // code
            tx.commit();  
        } catch (Exception e) {  
            tx.rollback();  
        } finally {  
            em.close();  
        }        
        emf.close();  
    }
}
```

- JPA는 항상 `EntityManagerFactory`를 만들어야 한다.
	- **데이터베이스당 하나만 만들어서 애플리케이션 전체에서 공유**
	- 위의 설정 파일에서 설정한 `persistence-unit`의 속성인 `name`의 인자로 "hello"를 보내서 해당 이름을 가진 설정 정보를 읽어와서 만들게 된다.
- 고객의 요청이 들어올 때마다 `EntityManager`를 통해서 작업을 해야한다.
	- **절대 쓰레드간에 공유를 하지 않는다. (사용하고 버려야 한다.)**
- **JPA의 모든 작업은 트랜잭션 안에서 일어나야 한다.**
- `try-catch-finally`를 사용한 이유는 혹시 **롤백이 되더라도 데이터베이스 커넥션이 반환**이 되기 때문이다.
	- 리소스 반환이 안됐을 경우 같은 커넥션 사용 시 잔여 데이터가 남아있을 수도 있음
	- `em.close()`
- WAS가 내려갈 때 `EntityManagerFactory`를 닫아줘야 한다.
	- 커넥션 풀링, 리소스의 릴리즈를 위함
	- `emf.close()`


### 저장

`em.persist()`를 사용해서 데이터를 데이터베이스에 영속화 시킨다.
- 괄호 안에는 저장시킬 데이터를 넣는다.
- 커밋 전까지는 아직 데이터베이스에 완전히 저장된 것이 아님.

```java
			// code
			Member member = new Member();
			member.setId(1L);
			member.setName("HelloA");
			em.persist(member);
```


### 조회

`em.find()`를 통해서 데이터베이스에 있는 데이터를 찾아온다.
- 첫 번째 인자는 반환할 데이터의 타입이다.
- 두 번째 인자는 찾으려는 데이터의 PK 정보이다.

```java
			// code
			Member findMember = em.find(Member.class, 1L);
```


### 수정

별다른 메서드 없이 조회 후 setter로 간단하게 수정할 수 있다.
- JPA를 통해서 엔티티를 가져오면 해당 엔티티는 JPA가 관리하게 된다. 
- 트랜잭션 커밋 시점에 데이터의 변경 사항을 체크해서 뭔가 바뀌었다면 `update` 쿼리를 날리게 된다.

```java
			// code
			Member findMember = em.find(Member.class, 1L);
			findMember.setName("HelloJPA");
			// 데이터가 바뀌었기 때문에 자동으로 update문을 만들어서 날림    
```


### JPQL 맛보기

간단하게 JPQL 코드를 보고 어떻게 사용하는지 확인해보자
- `em.createQuery()`를 통해서 `jpql`을 작성한다.
- 첫 번째 인자는 실행할 `jpql`문을 작성한다.
- 두 번째 인자는 클래스 타입을 지정한다.

```java
			// code
			List<Member> findMembers = em.createQuery(
				"select m from Member m", Member.class
			).getResultList();    
```

예제 코드처럼 모든 데이터를 가져오는 경우에는 `.getResultList()`로 결과값을 리스트에 저장할 수 있다. 또한 페이지네이션도 할 수 있는데 `.setFirstResult()`로 시작 값을, `.setMaxResults()`로 최대 값을 지정할 수 있다.
- 시작 값은 0 부터 시작한다.
- 최대 값은 개수로 판단한다.
	- 만약 `.setFirstResult(0).setMaxResults(1)`인 경우 0 번째 값 1개만 List에 저장하게 된다.