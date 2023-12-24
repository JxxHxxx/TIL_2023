
`@Transactional` 애노테이션은 스프링 컨테이너가 로딩된 직후, 초기화된다. 다르게 말하면 컴포넌트 의존성을 주입하는 시점에서는 `@Transactional` 을 통해 프록시 객체가 생성된 상태가 아니라는 의미이다.

아래와 같이 코드를 작성한 뒤, 빈 테스트 케이스를 작성해서 실행해보면 로그를 확인해 볼 수 있다.

```
@PostConstruct  
@Transactional  
public void post_construct() {  
    boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();  
    log.info("Hello init @PostConstruct tx active={}", isActive);  
}  
  
@EventListener(ApplicationReadyEvent.class)  
@Transactional  
public void application_ready() {  
    boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();  
    log.info("Hello init ApplicationReadyEvent tx active={}", isActive);  
}
```


![[Pasted image 20231224151119.png]](images/Pasted%20image%2020231224151119.png)

`@PostConstruct` 가 선언된 메서드가 먼저 호출된 뒤, 애플리케이션이 로딩된다. (`JVM running`)
이후, 트랜잭션을 만드는 것을 확인할 수 있다.
