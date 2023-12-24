
AOP 도입 이후 트랜잭션 코드와 비즈니스 로직을 분리하였다. 

즉 `Controller` -> `Service$Proxy` -> `Service` 흐름으로 메서드가 호출되게 된다. 이로 인해 이로 인해 프록시 내부 호출 문제를 주의해야 한다.



트랜잭션 관련 코드를 처리하는 `Service$Proxy` 에서는 `Serivce`  클래스 혹은 내부 메서드에 `@Transactional` 선언 유무에 따라 트랜잭션 처리를 한다. 이후 비즈니스 로직은 `Serivce` 에서 처리한다.

이로 인해 한 클래스 내부 `@Transactional` 선언 되지 않은 메서드에서 `@Transactional` 이 선언된 메서드를 호출하면 프록시 객체는 이를 감지할 수 없다. 이해가 되지 않아도 된다. 이제 예제 코드를 살펴보자.


```
@Slf4j  
public class CallService {  
  
    public void external() {  
        log.info("call external"); 
        printTxInfo(); 
        this.internal();  
    }  
  
    @Transactional  
    public void internal() {  
        log.info("call internal");  
        printTxInfo();
    }
    
	private void printTxInfo() {  
	    boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();  
	    log.info("tx active={}", txActive);  
	}
```


위에서 언급한대로 `external()` 에서 `@Transactional` 이 선언된 `internal()` 메서드를 호출하고 있다. 추가로 현재 트랜잭션이 활성화 되어있는지 확인하기 위해 `printTxInfo()` 출력 메서드를 하나 추가하였다.

테스트 케이스를 통해 실제로 트랜잭션이 활성화 되어 있는지 확인해보자.

```
@Test  
void external_call() {  
    callService.external();  
}
```

결과

![[Pasted image 20231224142745.png]](images/Pasted%20image%2020231224142745.png)

다음 장에서는 이러한 프록시 내부 호출 문제를 해결하는 방법에 대해 알아보도록 하겠다.
