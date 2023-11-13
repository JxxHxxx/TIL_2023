
배치 처리는 특성상 **상태**를 가지고 있다. 현재 어떤 스텝이 실행되고 잇는지, 해당 스텝이 처리한 레코드 개수도 알아야 한다. 뿐만 아니라 이전에 실패한 처리를 다시 시작하는데도 상태가 이용된다.

앞서 `JobExecution` 이 어떻게 실제 잡 실행 시도를 나타내는지 살펴봤다. `JobExecution` 은 잡이나 스텝이 진행될 때 변경되고 잡 상태는 `JobExecution` 의 `ExecutionContext` 에 저장된다.

`JobExecutionContext` 는 `key : value` 보관하는 객체에 불과하다. 하지만 **안전하게** 상태를 보관한다. 안전한 이유는 `ExecutionContext`가 담고 있는 모든 것이  **`JobRepository` 에 저장되기 때문이다.**



정리하면 `ExecutionContext` `Job` , `Step` 의 상태들을 저장한다. 사용 방법은 쉽다. `Tasklet` 에 관리하고 싶은 상태 값들을 저장하면 된다.


사용 예시

```
@Override  
public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {  
    String name= (String) chunkContext.getStepContext()  
            .getJobParameters()  
            .get("name");  
    // JobExecutionContext 에 상태 저장
    ExecutionContext executionContext = chunkContext.getStepContext()  
            .getStepExecution()  
            .getJobExecution()  
            .getExecutionContext();  
  
    executionContext.put("user.name", name);  
    System.out.println(String.format(HELLO_WORLD, name));  
    return RepeatStatus.FINISHED;  
}
```


이렇게 다시 빌드를 하고 배치 애플리케이션에 name 파라미터를 지정해서 실행해보자.

```
java -jar .\spring_batch_guide-0.0.1-SNAPSHOT.jar name=jxxhxxx
```



```
SELECT * FROM batch_job_execution_context bjec
WHERE JOB_EXECUTION_ID = 31;
```


`SHORT_CONTEXT` 컬럼의 내용이 바로 `ExecutionContext` 를 `JSON` 형식으로 표현한 값이다. 마지막 컬럼 `SERIALIZED_CONTEXT` 는 잡이 실행중이거나 실패한 경우에만 채워진다.

![[Pasted image 20231111193841.png]](../images/Pasted%20image%2020231111193841.png)



##### `JobExecution` 1개는 N개의 ExecutionContext 을 가질 수 있다. 
---
앞서 `ChunkContext` 를 통해 `Job` 의 `ExecutionContext` 를 가져왔다. 하지만 `ExecutionContext` 는 `Step` 에도 있을 수 있다. 그리고 `Job` 은 1개 이상의 `Step` 을 가질 수 있다. 따라서 하나의 `JobExecution` 에는 복수 개의 `JobExecution` 이 존재할 수 있다.


참고로 `StepExecution` 에서 해당 `ExecutionContext` 를 가져올 수 있다. 

```
ExecutionContext stepExecutionContext = chunkContext.getStepContext()  
        .getStepExecution()  
		.getExecutionContext();
```
