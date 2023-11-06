
##### Named 쿼리 - 정적 쿼리
---
- 미리 정의해서 이름을 부여해두고 사용하는 JPQL
- 정적 쿼리
- 어노테이션, XML에 정의
- **애플리케이션 로딩 시점에 초기화 후 재사용**
- **애플리케이션 로딩 시점에 쿼리를 검증**


*Named Query 예시*

---


*엔티티*

```
@Entity  
@NamedQuery(  
        name = "Member.findByName",  
        query = "select m from Member m where m.name =:name"  
)  // 만약 query 문법에 오류가 있으면 애플리케이션 로딩 시점에 에러 발생
public class Member {
```


*사용처*

```
List<Member> members = em.createNamedQuery("Member.findByName", Member.class)  
        .setParameter("name", "재헌")  
        .getResultList();
```


*쿼리 문법에 오류가 있을 때 발생하는 에러*

```
@Entity  
@NamedQuery(  
        name = "Member.findByName",  
        query = "select m from MemberQQA m where m.name =:name"  
) 
public class Member {
```

![[Pasted image 20231105182757.png]]


##### spring data jpa - Named Query

---

`spring data jpa` 라이브러리를 이용하면 더 깔끔하게 네임드 쿼리를 사용할 수 있다. 순수 JPA를 사용할 때는 엔티티 클래스에 네임드 쿼리가 들어갔다.

`spring data jpa` 를 사용한다면 `dao` 클래스에서 네임드 쿼리를 사용할 수 있다. 

```
public interface MemberRepository extend JpaRepository<Member, Long> {

	@Query("select m from m Member m where m.id=:id")
	Optional<Member> findById(Long id);
}
```