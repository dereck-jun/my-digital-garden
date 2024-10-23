---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 조인

조인에 대한 기본 지식이 선행되어야 함 (참고: [[template/DataBase/MySQL/join\|SQL 기본 문법 JOIN]])


### 내부 조인

`inner join`으로 써도 상관 없지만 그냥 `join`으로 사용해도 된다.

```java
String sql = "select m from Member m inner join m.team t";  
List<Member> resultList = em.createQuery(sql, Member.class)  
        .getResultList();
```

```cmd
Hibernate: 
    /* select
        m 
    from
        Member m 
    inner join
        m.team t */ select
            m1_0.id,
            m1_0.age,
            m1_0.team_id,
            m1_0.username 
        from
            Member m1_0 
        join
            Team t1_0 
                on t1_0.id=m1_0.team_id
```

현재 `Member` 엔티티에 `Team`과의 관계를 다대일이지만 `fetch` 타입을 기본 값인 `EAGER`가 아닌 `LAZY`로 설정했기 때문에 `select` 문이 1개만 나온다.

만약 `fetch`를 기본 값으로 두었다면 `select` 문이 2개 나오게 된다.


### 외부 조인

`<left | right> outer join`으로 써도 되지만 `<left | right> join`이라고 써도 된다.

`full outer join`의 경우 JPQL 스펙 상 지원하지 않기 때문에 네이티브 쿼리를 사용해야 한다.

```java
String sql = "select m from Member m left join m.team t";  
List<Member> resultList = em.createQuery(sql, Member.class)  
        .getResultList();
```

```cmd
Hibernate: 
    /* select
        m 
    from
        Member m 
    left join
        m.team t */ select
            m1_0.id,
            m1_0.age,
            m1_0.team_id,
            m1_0.username 
        from
            Member m1_0
```


### 세타 조인

따로 `join`을 적을 필요가 없다.

```java
String sql = "select m from Member m, Team t";  
List<Member> resultList = em.createQuery(sql, Member.class)  
        .getResultList();
```

```cmd
Hibernate: 
    /* select
        m 
    from
        Member m,
        Team t */ select
            m1_0.id,
            m1_0.age,
            m1_0.team_id,
            m1_0.username 
        from
            Member m1_0,
            Team t1_0
```

버전에 따라 실행 결과에 `cross join`으로 명시가 되거나 위의 결과처럼 나오지 않을 수도 있다. 하지만 실행 자체는 똑같이 된 것이니 크게 상관하지 않아도 된다.


## 조인 - ON 절

ON 절을 활용한 조인은 JPA 2.1 부터 지원한다. 
1. 조인 대상 필터링
	- 조인할 때 미리 조인 대상을 필터링 할 수 있음
2. 연관 관계 없는 엔티티 외부 조인(하이버네이트 5.1 부터 지원)
	- `where m.username = t.name`처럼 관계 없는 것을 외부 조인할 수 있음
	- 이전엔 내부 조인만 지원 했었음


### 조인 대상 필터링

회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

```java
String sql = "select m, t from Member m left join m.team t on t.name = 'A'";
```

```sql
SELECT m.*, t.* from
Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'
```


### 연관관계 없는 엔티티 외부 조인

회원의 이름과 팀의 이름이 같은 대상 외부 조인

```java
String sql = "select m, t from Member m left join Team t on m.username = t.name";
```

```sql
SELECT m.*, t.* FROM
Member m LEFT JOIN Team t ON n.username = t.name
```

