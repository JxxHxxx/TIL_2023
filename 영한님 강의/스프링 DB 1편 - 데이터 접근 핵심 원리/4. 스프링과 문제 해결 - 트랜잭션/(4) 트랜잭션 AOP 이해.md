
- 지금까지 트랜잭션을 편리하기 처리하기 위해서 트랜잭션 추상화, 템플릿을 도입했다. 많은 문제를 해결했지만 서비스 계층에 순수한 비즈니스 로직을 남긴다는 목표는 아직 달성하지 못했다.
- 이럴 때 스프링 AOP를 통해 프록시를 도입하면 문제를 깔끔하게 해결할 수 있다.

메서드 혹은 클래스 레벨에 `@Transactional` 를 선언하면 스프링이 이를 감지해서 프록시 객체를 생성한다

```
// MemberService.class 
@Transactional  
public void accountTransfer(String fromId, String toId, int money) throws SQLException {  
    bizLogic(fromId, toId, money);  
}
```


프록시 기술 중 하나인 `CGLIB`를 통해 프록시 객체가 생성되는 것을 로그로도 확인할 수 있다.

![[Pasted image 20231008172300.png]]

프록시는 간단하게 어떠한 일을 대신 처리하는 대리자로 이해하면 편하다. 이렇게 만들어진 프록시 객체가 트랜잭션과 관련된 작업들을 처리하고 실제 비즈니스 로직은 진짜 `MemberService` 가 처리하게 된다. 

이제 우리는 `@Transactional` 선언 하나로 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 분리할 수 있게 되었다. 

*참고*

- 외부에서 접근할 수 있는 `public` 클래스, 메서드만 AOP 대상이 된다.


