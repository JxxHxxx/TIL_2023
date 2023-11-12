

`JobLauncher` 인터페이스 : 잡을 실행시키는 인터페이스

```
public interface JobLauncher {  
  
public JobExecution run(Job job, JobParameters jobParameters) throws   
		 JobExecutionAlreadyRunningException,  
         JobRestartException, 
         JobInstanceAlreadyCompleteException, 
         JobParametersInvalidException;  
  
}
```

구현체로는 `SimpleJobLauncher` 가 기본적으로 주입된다. 잡 실행과 관련된 대부분의 요구 사항을 만족한다고 한다. 잡의 실행이 기존 잡 인스턴스의 일부인지 새로운 잡의 일부인지를 판별해 그에 맞는 동작을 한다.

다만 `SimpleJobLauncher` 는 전달받은 `JobParameters` 조작을 지원하지 않는다. 따라서 잡에 `JobParameterIncrementer` 를 사용해야 한다면, 해당 파라미터가 `SimpleJobLauncher` 로 전달되기 전에 적용해야 한다.


추가로 `application.yml` 애플리케이션 실행 시 배치가 시작되지 않도록 값을 저정해주자.

```
spring:  
  batch:  
    job:  
      enabled: false
```


REST API 를 호출하기 위해 컨트롤러 레이어 클래스를 만들어주자.


```
@RestController  
@RequiredArgsConstructor  
public class JobLaunchingController {  
  
    private final JobLauncher jobLauncher;  
    private final ApplicationContext context;  
  
    @PostMapping(path = "/run")  
    public ExitStatus runJob(@RequestBody JobLaunchRequest request) throws Exception {  
        Job job = this.context.getBean(request.getName(), Job.class);  
  
        return this.jobLauncher.run(job, request.getJobParameters())  
                .getExitStatus();  
    }  
}
```


`JobLaunchRequest` 는 잡 실행에 필요한 파라미터를 넘겨 받을 dto 성격의 클래스이다. ``

```
@Getter  
@Setter  
public class JobLaunchRequest {  
    private String name;  
    private Properties jobParameters;  
  
    public JobParameters getJobParameters() {  
        Properties properties = new Properties();  
        properties.putAll(this.jobParameters);  
        return new JobParametersBuilder(properties)  
                .toJobParameters();  
    }  
}
```


잡과 스텝은 아래와 같이 구성하자. 앞서 말했지만 `incrementer` 메서드를 호출하더라도 이미 파라미터가 `SimpleJobLauncher` 에 넘어갔기 때문에 동작하지 않는다.

```
@Bean  
public Job job1() {  
    return this.jobBuilderFactory.get("job1")  
            .incrementer(new RunIdIncrementer()) // 동작 안함  
            .start(step1())  
            .build();  
}  
  
@Bean  
public Step step1() {  
    return stepBuilderFactory.get("step1")  
            .tasklet((contribution, chunkContext) -> {  
                System.out.println("step 1 ran today!");  
                return RepeatStatus.FINISHED;  
            }).build();  
}
```


이제 완성이 됐다. API 를 호출해보면 정상적으로 실행된걸 확인할 수 있다. 하지만 잡 파라미터 증가 처리를 하지 않았기 때문에 재호출할 수 없다,

![[Pasted image 20231113001742.png]]


`JobParametersBuilder` 가 제공하는 `getNextJobParameters` 메서드를 이용하면 쉽게 파라미터를 증가시킬 수 있다. 더 자세히는 `RunIdIncrementer` 를 활성화시키는 작업이다.

```
@PostMapping(path = "/v2/run")  
public ExitStatus runJob2(@RequestBody JobLaunchRequest request) throws Exception {  
    Job job = this.context.getBean(request.getName(), Job.class);  
    JobParameters jobParameters = new JobParametersBuilder(request.getJobParameters(), this.jobExplorer)  
            .getNextJobParameters(job)   구체적으로는
            .toJobParameters();  
  
    return this.jobLauncher.run(job, jobParameters).getExitStatus();  
}
```

실제로 같은 API를 두 번 호출해도 잘 실행되는걸 확인할 수 있다. 

![[Pasted image 20231113002333.png]]


잡 실행 파라미터를 보면 `run.id` 값이 증가하는걸 볼 수 있다.

```
select * from batch_job_execution_params
```

![[Pasted image 20231113002415.png]]
