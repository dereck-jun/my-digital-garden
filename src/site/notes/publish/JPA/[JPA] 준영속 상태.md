---
{"dg-publish":true,"permalink":"/publish/jpa/jpa/"}
---

## 준영속 상태

- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)된 것이다. 한마디로 그냥 다 빼버리는 것이다.
- 준영속 상태의 경우 영속성 컨텍스트가 제공하는 기능을 사용 못한다.

```java
Member findMember = em.find(Member.class, 101L);  // ID = 101L 객체가 영속 상태로 변경
findMember.setName("ASDF");  // 더티 체킹으로 update 쿼리가 날라갈 예정
em.detach(findMember);  // 이었으나 준영속 상태로 변경 (JPA에서 관리 안함)

tx.commit();  // select 문만 전송되고 update 문은 전송되지 않음
```


## 준영속 상태로 만드는 방법

- `em.detach()` - 특정 엔티티만 준영속 상태로 전환
- `em.clear()` - 영속성 컨텍스트를 완전히 초기화
- `em.close()` - 영속성 컨텍스트 종료