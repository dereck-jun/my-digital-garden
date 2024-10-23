---
{"dg-publish":true,"permalink":"/publish/jpa/jpa-api/"}
---

JPA는 페이징을 다음 두 API로 추상화했다.

- setFirstResult(int startPosition): 조회 시작 위치 (시작 값: 0)
- setMaxResults(int maxResult): 조회할 데이터 수

이때 실제 DB에서 동작할 쿼리는 방언에 따라 달라진다.

사용하는 방법은 다음과 같다.

```java
for (int i = 0; i < 100; i++) {  
    Member member = new Member();  
    member.setUsername("user" + (i + 1));  
    member.setAge(i + 1);  
    em.persist(member);  
}  
  
em.flush();  
em.clear();  
  
List<Member> resultList = em.createQuery(
			"select m from Member m order by m.age desc", Member.class)  
        .setFirstResult(0)  
        .setMaxResults(7)  
        .getResultList();  
  
  
System.out.println("resultList.size() = " + resultList.size());  
for (Member member : resultList) {  
    System.out.println("member = " + member.toString());  
}
```

```cmd
Hibernate: 
    /* select
        m 
    from
        Member m 
    order by
        m.age desc */ select
            m1_0.id,
            m1_0.age,
            m1_0.team_id,
            m1_0.username 
        from
            Member m1_0 
        order by
            m1_0.age desc 
        offset
            ? rows 
        fetch
            first ? rows only
resultList.size() = 7
member = Member{id=100, username='user100', age=100}
member = Member{id=99, username='user99', age=99}
member = Member{id=98, username='user98', age=98}
member = Member{id=97, username='user97', age=97}
member = Member{id=96, username='user96', age=96}
member = Member{id=95, username='user95', age=95}
member = Member{id=94, username='user94', age=94}
```