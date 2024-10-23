---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-jqpl/"}
---

## 조건식

### 기본 CASE 식

기본 CASE 식은 컨디션에 대한 조건을 넣을 수 있다.

```java
em.createQuery("select " +  
        "case when m.age <= 10 then '학생 요금' " +  
        "when m.age >= 60 then '경로 요금' " +  
        "else '일반 요금' " +  
        "end " +  
        "from Member m", Member.class);
```


### 단순 CASE 식

단순 CASE 식은 정확하게 매칭이 되었을 때만 행동을 하게 한다.

```java
em.createQuery("select " +  
        "case t.name " +  
        "when '팀A' then '인센티브 120%' " +  
        "when '팀B' then '인센티브 125%' " +  
        "else '인센티브 없음' " +  
        "end " +  
        "from Team t", Team.class);
```


### COALESCE

하나씩 조회해서 `null`이 아니면 반환하게 한다.

```java
em.createQuery("select coalesce(m.username, '이름 없는 회원') from Member m");
```

조회해서 `null`이 아니면 `m.username` 값을 반환하고, `null`이면 `이름 없는 회원`을 반환


### NULLIF

두 값이 같으면 `null`을 반환하고, 다르면 첫 번째 값을 반환한다.

```java
em.createQuery("select nullif(m.username, '관리자') from Member m");
```

조회해서 `m.username`이 `관리자`면 `null`을 반환하고, `관리자`가 아니면 `m.username`의 값을 반환


## 기본 함수

### CONCAT

`concat()` 안의 내용을 합친다.

```java
em.createQuery("select concat(m.firstname, m.lastname) from Member m", Member.class);
```

`m.firstname`과 `m.lastname`을 합쳐서 반환

### SUBSTRING

문자열을 자른 결과 값을 반환

```java
em.createQuery("select substring(m.name, 1, 2) from Member m", String.class);
```

`m.name`의 첫 번째부터 2개를 잘라낸 값을 반환


### TRIM

공백을 제거한다.

LTRIM, RTRIM 옵션으로 해서 사용 가능

```java
em.createQuery("select TRIM('   hi   ') from Member m", String.class);
```


### UPPER, LOWER

대소문자 변경해서 반환

```java
em.createQuery("select upper('creATE') from Member m", String.class);

em.createQuery("select lower('creATE') from Member m", String.class);
```


### LENGTH

문자의 길이 반환

```java
em.createQuery("select length(m.name) from Member m", Integer.class);
```


### LOCATE

찾으려는 문자가 있는 위치를 반환

```java
em.createQuery("select locate('A', 'creATE') from Member m", Integer.class);
```



### SIZE

컬렉션의 크기를 반환

```java
em.createQuery("select size(t.members) from Team t", Integer.class);
```


### INDEX

컬렉션 내 특정 요소의 위치 값을 반환한다.

`index()`는 리스트의 요소 위치를 반환하지만, 중간에 요소가 삭제되면 인덱스가 비어 있을 수 있다. 

이로 인해 쿼리 결과에 `null` 값이 포함될 수 있으므로 사용하지 않는 것이 좋다.

```java
em.createQuery("select index(t.members) from Team t", Integer.class);
```



## 사용자 정의 함수

사용자 정의 함수를 만드는 방법은 다음과 같다.

### 사용자 정의 함수 생성

먼저 클래스를 만들고 사용할 데이터베이스의 방언을 상속 받는다.

```java
public class MyH2Dialect extends H2Dialect {\
}
```

사용하려는 방언을 상속받은 뒤 생성자에서 `registerFunction()`을 사용해서 등록해주면 된다.

```java
public class MyH2Dialect extends H2Dialect {\
	public MyH2Dialect() {
		registerFunction("group_concat", 
			new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
	}
}
```

`new StandardSQLFunction()`의 경우 외워서 사용하기엔 무리가 있으므로 소스 코드를 열어서 살펴본 뒤 사용하면 된다.

마지막으로 데이터베이스 방언을 해당 클래스로 변경해주면 된다.

```xml
<properties>
	<property name="hibernate.dialect" value="dialect.MyH2Dialect"/>
</properties>
```


### 사용자 정의 함수 호출

사용자 정의 함수를 호출할 땐 `function()` 안에 사용할 함수의 이름을 넣어주고 사용하면 된다.

```java
em.createQuery(
	"select function('group_concat', m.username) from Member m", String.class);
```

