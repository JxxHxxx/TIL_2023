
##### 영속성 전이 : CASCADE

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다.
- 예시) `@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)`
- 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 **편리함**을 제공하는 기능이다.


```
Member memberA = new Member("helloA");  
Member memberB = new Member("helloB");  
Team team1 = new Team("team1");  

memberA.setTeam(team1);  // 연관 관계 설정
memberB.setTeam(team1);  // 연관 관계 설정
team1.getMembers().addAll(List.of(memberA, memberB));  // 영속성 전이를 위해...

em.persist(team1);
```


###### CASCADE 종류
---

- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제

3가지가 주요하다. `PERSIST` 는 영속화 할 때만 연관된 엔티티도 함께 영속상태로 만든다.  `REMOVE` 는 삭제할 때 연관된 엔티티도 함께 삭제한다. 상황에 따라 다르겠지만 `PERSIST` 는 유용하지만 `REMOVE` 는 고려해봐야 한다. 

그 이유는 다음과 같다.

삭제할 때 만약 다른 곳에서 연관된 엔티티를 참조하고 있다면 문제가 생길 수 있다. 다른 측면에서는 실무에서는 데이터를 삭제하기도 하지만 논리 삭제가 우선 순위가 높다.



##### 고아 객체
---
- 고아 객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- 예시 `orphanRemoval = true`

고아 객체(`orphanRemoval`) 또한 삭제 시 자식 엔티티까지 함께 삭제한다. `CascadeType.REMOVE` 처럼 동작한다는 의미이다.
