
##### Fetch Join
---
- SQL 조인 X
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 한 번의 SQL로 함께 조회하는 기능

- 사용 이유 : N + 1 문제 예방


#### 다대일 페치 조인
---

*사용 방법*

```
String query = "select m from Member m (inner/outer) join fetch m.team"
```



##### 컬렉션 페치 조인
---
일대다 관계에서 사용한다. 일대다 관계에서 페치 조인을 가져오면 **다**엔티티 개수 만큼 데이터가 늘어나게 된다.

예를 들어 한 팀에 멤버가 4명 있으면 페치 조인으로 한 팀을 조회하면 레코드 4줄을 가져온다.

*예시*

```
String query = "select t from Team t join fetch t.members m";  
  
List<Team> teams = em.createQuery(query, Team.class)  
        .getResultList();  
  
teams.forEach(t -> log.info("team {} member {}", t, t.getMembers()));
```


결국 데이터베이스 입장에서는 아래와 같이 가져오게 된다. 

![[Pasted image 20231102205545.png]]

그런데 여기서 JPA는 4줄을 가져오는지 알 수 없다. 그래서 일단 조회한 쿼리를 영속성 컨텍스트에 올린다. 다시 말해 Team 엔티티 4개를 영속성 컨텍스트에 올린다.

다만 PK가 동일하기 때문에 컬렉션(`teams`)에 담긴 요소들은 모두 동일한 엔티티를 참조하게 된다.

그래서 로그도 아래처럼 4번 찍히게 된다.

```
20:57:39.247 [main] INFO com.jpql.hello.JpaMain - team Team(id=7, name=teamA) member [Member(id=6, name=재헌, age=30, memberType=ADMIN), Member(id=8, name=null, age=11, memberType=USER), Member(id=9, name=윤정, age=19, memberType=USER), Member(id=10, name=탁, age=61, memberType=USER)]

20:57:39.277 [main] INFO com.jpql.hello.JpaMain - team Team(id=7, name=teamA) member [Member(id=6, name=재헌, age=30, memberType=ADMIN), Member(id=8, name=null, age=11, memberType=USER), Member(id=9, name=윤정, age=19, memberType=USER), Member(id=10, name=탁, age=61, memberType=USER)]

20:57:39.277 [main] INFO com.jpql.hello.JpaMain - team Team(id=7, name=teamA) member [Member(id=6, name=재헌, age=30, memberType=ADMIN), Member(id=8, name=null, age=11, memberType=USER), Member(id=9, name=윤정, age=19, memberType=USER), Member(id=10, name=탁, age=61, memberType=USER)]

20:57:39.277 [main] INFO com.jpql.hello.JpaMain - team Team(id=7, name=teamA) member [Member(id=6, name=재헌, age=30, memberType=ADMIN), Member(id=8, name=null, age=11, memberType=USER), Member(id=9, name=윤정, age=19, memberType=USER), Member(id=10, name=탁, age=61, memberType=USER)]
```


이 문제를 해결하려면 JPA에서 지원하는 `distinct` 를 사용하면 된다. 실제 쿼리도 `distinct` 가 나가긴 하지만 추가로 애플리케이션 레벨에서  같은 식별자를 가진 `t` 즉 `Team` 엔티티의 중복을 제거한다.

```
String query = "select distinct t from Team t join fetch t.members m";  
```


*실제 나가는 쿼리*

```
select
	distinct team0_.id as id1_3_0_,
	members1_.id as id1_0_1_,
	team0_.name as name2_3_0_,
	members1_.age as age2_0_1_,
	members1_.memberType as memberty3_0_1_,
	members1_.name as name4_0_1_,
	members1_.TEAM_ID as team_id5_0_1_,
	members1_.TEAM_ID as team_id5_0_0__,
	members1_.id as id1_0_0__ 
from
	Team team0_ 
inner join
	Member members1_ 
		on team0_.id=members1_.TEAM_ID
```


*로그 결과*

```
21:12:32.867 [main] INFO com.jpql.hello.JpaMain - team Team(id=7, name=teamA) member [Member(id=6, name=재헌, age=30, memberType=ADMIN), Member(id=8, name=null, age=11, memberType=USER), Member(id=9, name=윤정, age=19, memberType=USER), Member(id=10, name=탁, age=61, memberType=USER)]
```




정리하면 

1. 실제 SQL DISTINCT 쿼리 나감
2. 애플리케이션에서 동일한 식별자를 가진 엔티티 중복 제거



