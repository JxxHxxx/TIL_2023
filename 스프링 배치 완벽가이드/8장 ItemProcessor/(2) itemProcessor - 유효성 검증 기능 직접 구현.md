
이 장은 조금 혼란스러울 수 있지만 2가지를 배우고 가야 한다.
- 유효성 검증 기능 직접 구현하는 방법
- `ExecutionContext` 및 `ItemStreamSupport` 의 메서드 호출 사이클


앞서 우리는 `BeanValidatingItemProcessor` 를 통해 간편하게 유효성 검증기를 구현했다. 이제 요구 사항이 변경됐다고 해보자. `LastName` 이 고유해야 한다고 하면 어떻게 해야할까? 

(여기서 설명하는 방식을 적용하면 `JSR-303` 검증은 제외된다.)

이를 위해서는 `BeanValidatingItemProcessor` -> `ValidatingItemProcessor` 로 변경하고, 직접 구현한 Validator 인터페이스 구현체를 주입해야 한다.

데이터셋 내에서 `lastName` 필드의 값이 고유해야 한다고 하자. 레코드가 해당 사항을 준수하는지 확인하기 위해서는 `lastName` 을 추적하는 상태를 가진 유효성 검증기(`stateful`)를 구현한다.

추가로 재시작 시에도 상태를 유지하려면 유효성 검증기는 `ItemStreamSupport` 를 상속해 `ExecutionContext` 에  각 커밋과 `lastName` 을 저장해야 한다.

아래는 예시 코드이다.

```
@Slf4j  
public class UniqueLastNameValidator extends ItemStreamSupport implements Validator<CustomerV5> {  
  
    private Set<String> lastNames = new HashSet<>();  
  
    @Override  
    public void validate(CustomerV5 value) throws ValidationException {  
        if (lastNames.contains(value.getLastName())) {  
            throw new ValidationException("Duplicate last name was found: " + value.getLastName());  
        }  
  
        this.lastNames.add(value.getLastName());  
    }  
  
    /**  
     * processor 내 한 번만 호출됨  
     */  
  
    @Override  
    public void open(ExecutionContext executionContext) {  
        log.info("call open");  
        String lastNames = getExecutionContextKey("lastNames");  
  
        if (executionContext.containsKey(lastNames)) {  
            this.lastNames = (Set<String>) executionContext.get(lastNames);  
        }  
    }  
  
    /**  
     * 트랜잭션이 커밋되면 청크당 한 번 호출됨  
     */  
  
    @Override  
    public void update(ExecutionContext executionContext) {  
        log.info("call update executionContext chunk count : {}", executionContext.get("customerItemReaderV5.read.count"));  
        Iterator<String> iterator = lastNames.iterator();  
        HashSet<String> copiedLastNames = new HashSet<>();  
  
        while (iterator.hasNext()) {  
            copiedLastNames.add(iterator.next());  
        }  
  
        executionContext.put(getExecutionContextKey("lastNames"), copiedLastNames);  
    }  
}
```


구성 클래스에서 추가된 부분은 아래와 같다.


```
@Bean  
public Step copyFileStepV5() {  
    return stepBuilderFactory.get("copyFileJobV2")  
            .<CustomerV5, CustomerV5> chunk(2)  
            .reader(customerItemReaderV5(null))  
            .processor(customerV5BeanValidatingItemProcessor()) 
            .writer(itemWriterV5())  
            .stream(uniqueLastNameValidator()) // restart 시 사용하기 위해
            .build();  
}

@Bean  
public ValidatingItemProcessor<CustomerV5> customerV5BeanValidatingItemProcessor() {  
    return new ValidatingItemProcessor<>(uniqueLastNameValidator());  
}  
  
@Bean  
public UniqueLastNameValidator uniqueLastNameValidator() {  
    UniqueLastNameValidator uniqueLastNameValidator = new UniqueLastNameValidator();  
    uniqueLastNameValidator.setName("uniqueLastNameValidator");  
    return uniqueLastNameValidator;  
}
```



##### `ExecutionContext`
---
이 내용은 책이 아닌 내가 스스로 파악한 내용이라 혹여나 고려하지 못한 부분이 있을 수도 있다. 그래도 로그와 DB를 모니터링해가며 파악한 부분이기에 크게 잘못된 부분은 없을 것이다.

`ExecutionContext` 은 우선 `ItemStream` 의 상태를 관리하는 객체이다. 상태(`state`)란 객체의 필드를 의미한다. 위 코드를 보면 `ItemStreamSupport` 를 상속받아 `ExecutionContext` 를 컨트롤 하는 것을 보았을 것이다.

`spring-batch` 의 장점 중 하나는 `ExecutionContext` 의 영속적인 관리를 돕는다는 것이다. 
앞서 실행한 Job의 step의 상태를 보도록 하겠다.

```
SELECT * FROM BATCH_STEP_EXECUTION_CONTEXT BSEC
WHERE STEP_EXECUTION_ID = 125;
```


![[Pasted image 20231212173550.png]]

여기에 SHORT_CONTEXT 컬럼의 값이 ExecutionContext 의 상태 정보를 볼 수 있다. 아래는 해당 값이다. 

```
{"@class":"java.util.HashMap",
"batch.taskletType":"org.springframework.batch.core.step.item.ChunkOrientedTasklet",
"customerItemReaderV5.read.count":6,
"uniqueLastNameValidator.lastNames":["java.util.HashSet,["Johnson","Minella","Hoover","Baxter","Gilbert","Heon"]],
"batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep"}
```

상태를 `HashMap` 객체에 담아두었다는 정보,
현재 스탭(`STEP_EXECUTION_ID` = 125)에서 읽은 레코드 수 `read.count = 6` 
그리고 우리가 추가로 설정한 `lastNames` 또한 확인할 수 있다. 이를 통해 값의 중복을 판단한다.

이렇게 `spring-batch` 를 활용하면 배치 처리의 중간 상태를 영속적으로 관리할 수 있다는 장점이 있다.
