
해결 방법은 간단하다. 내부 메서드를 다른 클래스로 분리하면 된다. 아래와 같이 클래스를 분리하면

`InternalService` 클래스에 대한 프록스 객체가 생성되고 `CallService.external()` 에서  `internal()` 을 호출할 때 순수 `InternalService` 객체가 아닌 프록시 객체를 먼저 거치게 된다.

```
@RequiredArgsConstructor  
public class CallService {  
  
    private final InternalService internalService;  
  
    public void external() {  
        log.info("call external");  
        printTxInfo();  
        internalService.internal();  
    }  
  
}  
  
public class InternalService {  
  
    @Transactional  
    public void internal() {  
        log.info("call internal");  
        printTxInfo();  
    }  
}
```


##### Public 메서드만 트랜잭션 적용
---
스프링 트랜잭션 AOP 기능은 `public` 메서드에만 트랜잭션을 적용하도록 설정되어 있다. 스프링에서 트랜잭션 적용에 대한 유연성을 제공한다고 보면 된다.