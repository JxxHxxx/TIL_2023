
#####  **1. 페치 조인 대상에는 별칭을 줄 수 없다.(권장하지 않음)**
---
하이버네이트는 가능하다. 하지만 가급적 사용하지 말자.

이유 : 페치 조인은 연관된 엔티티를 모두 가져오는 것이다. 그런데 별칭을 찍어서 연관된 엔티티의 일부만 컨트롤하는건 위험할 수 있다는 의미이다. **즉 아래와 같이 사용해서 연관된 엔티티 일부만 불러와서 컨트롤하지 말라는 것이다.**

```
select t From Team t join fetch t.members m where m.age > 10
```


##### 2. 둘 이상의 컬렉션은 페치 조인 할 수 없다.(권장하지 않음)
---

예전에 팀 프로젝트를 할 때 했던 기억이 있긴한데 나중에 추가로 정리해야 겠다. 이걸 JPA에서 막은 이유는 데이터의 정합성 및 성능 때문이다.

둘 이상의 컬렉션을 페치 조인 하는 순간 데이터가 1 : N : N 으로 불어난다. 

```
String query = "select t from Team t " +  
        "    join fetch t.members " +  
        "    join fetch t.extranetAccesses ";
```

현재 각 테이블의 레코드는 `Team` 1 , `Member` 4  `ExtranetAccesses`
2 개이다. 1 * 4 * = 8 개의 레코드를 조회한다. 

![[Pasted image 20231104114408.png]]

실제로 나가는 쿼리는 아래와 같다.
```
SELECT
    team0_.id as id1_4_0_,
    members1_.id as id1_1_1_,
    extranetac2_.EXTRANET_ACCESS_ID as extranet1_0_2_,
    team0_.name as name2_4_0_,
    members1_.age as age2_1_1_,
    members1_.memberType as memberty3_1_1_,
    members1_.name as name4_1_1_,
    members1_.TEAM_ID as team_id5_1_1_,
    members1_.TEAM_ID as team_id5_1_0__,
    members1_.id as id1_1_0__,
    extranetac2_.extranetAccessType as extranet2_0_2_,
    extranetac2_.name as name3_0_2_,
    extranetac2_.team_id as team_id4_0_1__,
    extranetac2_.EXTRANET_ACCESS_ID as extranet1_0_1__
FROM
    Team team0_
INNER JOIN
    Member members1_ ON team0_.id = members1_.TEAM_ID
INNER JOIN
    ExtranetAccess extranetac2_ ON team0_.id = extranetac2_.team_id

```


##### 3. 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
---

객체 그래프 상에서는 컬렉션을 페치 조인 할 수 없다. `team` : `Member` = `1` : `N`  이기 때문에 아래 쿼리는 컬렉션을 페치 조인하고 페이징 쿼리를 날리게 된다. 이렇게 되면  `TeamA` 에 속한 `Member` 가 5명이더라도 1명만 가져와야 하는데 이것은 객체 그래프 상 올바르지 못한 설계이다. 객체 그래프는 엔티티의 정보를 모두 가져와야 한다.

*페이징 API*

```
String query = "select t from Team t join fetch t.members m ";  
List<Team> teams = em.createQuery(query, Team.class)  
        .setFirstResult(0)  
        .setMaxResults(1)  
        .getResultList();
```


*실제로 나가는 쿼리*

쿼리를 보면 `limit` , `offset` 이 없다. 즉 모든 팀의 멤버를 가져온다.

```
select
	team0_.id as id1_4_0_,
	members1_.id as id1_1_1_,
	team0_.name as name2_4_0_,
	members1_.age as age2_1_1_,
	members1_.memberType as memberty3_1_1_,
	members1_.name as name4_1_1_,
	members1_.TEAM_ID as team_id5_1_1_,
	members1_.TEAM_ID as team_id5_1_0__,
	members1_.id as id1_1_0__ 
from
	Team team0_ 
inner join
	Member members1_ 
		on team0_.id=members1_.TEAM_ID
```


##### 해결 방법 1. N : 1에서 페이징
---

반면 `Member` : `Team` = `N` : `1` 이기 때문에 페이징 쿼리를 날리더라도 상관이 없다. 왜냐하면 `Member` 가 속한 `Team` 은 1개이기 때문에 컬렉션 페이징이 아니기 때문이다. 즉, `Member` 엔티티 단위로 페이징을 하게 된다.

```
String query = "select m from Member m join fetch m.team t ";  
List<Member> members = em.createQuery(query, Member.class)  
        .setFirstResult(0)  
        .setMaxResults(1)  
        .getResultList();
```


*실제로 나가는 쿼리*

```
select
	member0_.id as id1_1_0_,
	team1_.id as id1_4_1_,
	member0_.age as age2_1_0_,
	member0_.memberType as memberty3_1_0_,
	member0_.name as name4_1_0_,
	member0_.TEAM_ID as team_id5_1_0_,
	team1_.name as name2_4_1_ 
from
	Member member0_ 
inner join
	Team team1_ 
		on member0_.TEAM_ID=team1_.id limit ?
```


##### 해결 방법 2 : 배치 사이즈
---

```
String query = "select t from Team t";  
List<Team> teams = em.createQuery(query, Member.class)  
        .setFirstResult(0)  
        .setMaxResults(2)  
        .getResultList();
```

이런식으로 설정하고 배치 사이즈를 설정해서 컬렉션을 `in` 쿼리로 한 번에 가져오는 방법이 있다.
자세한 내용은 심화 편에서 다루도록 하겠다.

```
@Entity
public class Team {  
	//...
	
    @OneToMany(mappedBy = "team")  
    @BatchSize(size = 100)  // 배치 사이즈 설정
    private List<Member> members = new ArrayList<>();
```

