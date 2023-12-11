

##### 플랫 파일
---
배치 처리와 관련해서 플랫 파일이란 한 개 또는 그 상의 레코드가 포함된 특정 파일을 의미한다. 플랫 파일은 파일의 내용을 봐도 데이터의 의미를 알 수 없다는 점에서 XML 파일과 차이가 있다. 다시 말해 플랫 파일에는 데이터의 포맷, 데이터의 의미를 정의하는 메타데이터가 없다. 

*FlatFileItemReader*
스프링 배치에서 파일을 읽을 때 사용하는 컴포넌트이다. 

`FlatFileItemReader` 는 다시 메인 컴포넌트 두 개로 이뤄진다. 
1. 읽어들일 대상 파일을 나타내는 스프링의 `Resource`
2.  `LineMapper` 는 스프링 `JDBC` 에서 `RowMapper` 와 비슷한 역할을 한다. 스프링 `JDBC` 에서 `RowMapper` 를 사용하면 필드의 묶음을 나타내는 `ResultSet`을 객체로 매핑할 수 있다.
3. `LineMapper` 는 다시 `LineTokenizer` , `FieldSetMapper` 로 나뉜다.


##### `FlatFileItemReader`
---

`FlatFileItemReader` 구성 시에는 속성을 지정해야 한다. 아래는  `FlatFileItemReaderBuilder` 를 사용해 `FlatFileItemReader` 속성들을 지정하는 예시이다.

```
        return new FlatFileItemReaderBuilder<FieldSet>()  
                .name("fileItemReader")  
                .resource(new ClassPathResource(inputFileName))  
                .linesToSkip(1)  
                .skippedLinesCallback(line -> log.info("skipped line : {} ", line))
                .lineTokenizer(new DelimitedLineTokenizer()) 
                .fieldSetMapper(payFieldSetMapper)  
                .build();  
    }
```

이렇게 사용하기 위해서는 `FlatFileItemReader` 의 속성들이 어떤 역할을 하는지 알고 있어야 하니 몇 가지 정리를 해보도록 하자.


- `name` :  `ExecutionContext` 에 저장되는 값의 고유 키를 생성하는 데 사용한다.

스프링 배치 메타 테이블 내에서 `name` 에서 지정한 의미를 알 수 있게 된다.

```
SELECT * FROM BATCH_STEP_EXECUTION_CONTEXT
ORDER BY STEP_EXECUTION_ID DESC;
```

우선 `itemReader` 는  `chunk` 기반의 `step` 에서 동작하기에  `BATCH_STEP_EXECUTION_CONTEXT` 테이블에 정보가 존재하게 된다.

정보의 값을 살펴보면 아래와 같다. 

```
{"@class":
"java.util.HashMap",
"fileItemReader.read.count":99, // fileItemReader <- name 애트리뷰트로 지정된 값
"batch.taskletType":"org.springframework.batch.core.step.item.ChunkOrientedTasklet",
"batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep"
}
```

다르게 말하면 `name(helloWorld)` 라고 지정하게 되면 `helloWorld.read.count` 라고 표현되어 있을 것이다.

- `resource` : 읽은 대상, 즉 (`txt`, `csv` 등 확장자를 가진)리소스를 의미함
- `linesToSkip` : 잡을 실행할 때, 파일을 파싱하기 전에 파일 시작부터 몇 줄을 건너뛸 것인지 지정
- `skippedLinesCallback` : 줄을 건너뛸 때 호출되는 콜백 인터페이스, 건너뛴 모든 줄은 이 콜백으로 넘겨진다. 위 예제 `linesToSkip` 을 통해 첫번 쨰 줄이 건너 뛰어지게 되는데 해당 내용이 이 콜백으로 넘겨진다.


앞서 `LineMapper` 는 `lineToTokenizer` ,`FieldSetMapper` 컴포넌트로 나뉜다고 했다. `LineMapper` 속성을 지정해도 되지만 이를 둘로 나눠 구분지어도 된다.

- `lineToTokenizer` : 줄(`line`)을 파싱해 `FieldSet` 으로 만든다. 여기서 제공되는 `String` 은 파일에서 가져온 한 줄 전체를 나타낸다. 
- `FieldSetMapper` : `FieldSet`을 도메인 객체로 매핑한다. 위 코드에서는 `payFieldSetMapper` 를 사용해서 `FieldSet` -> `Pay` 라는 도메인 객체로 매핑한다.

아래는`FieldSetMapper` 구현체인 `payFieldSetMapper` 클래스 코드 일부이다. 이정도만 참고해도 `FieldSetMapper` 의 역할이 무엇인지 알 수 있을 것이다.

```
@Component  
public class PayFieldSetMapper implements FieldSetMapper<Pay> {  
  
    @Override  
    public Pay mapFieldSet(FieldSet fieldSet) throws BindException {  
        PayDto payDto = create(fieldSet.getValues());  
  
        OrderInformation orderInformation = new OrderInformation(payDto.orderNo, payDto.orderAmount, payDto.storeId);  
        PayInformation payInformation = new PayInformation(payDto.payerId, payDto.payAmount, payDto.vatAmount, payDto.payType, payDto.payStatus);  
  
        return new Pay(orderInformation, payInformation, payDto.createdDate, payDto.createdTime);  
    }
```

참고로 아래는 `LineMapper`  `FlatFileItemReader`  를 구현하는 방법이다.


```
@Bean  
@StepScope  
public FlatFileItemReader<FieldSet> flatFileItemReader(@Value("#{jobParameters['payFile']}") String inputFileName) throws IOException {  
    return new FlatFileItemReaderBuilder<FieldSet>()  
            .name("fileItemReaderV2")  
            .resource(new ClassPathResource(inputFileName))  
            .linesToSkip(1)  
            .skippedLinesCallback(line -> log.info("skipped line : {}", line))  
            .lineMapper(payLineMapper())  
            .build();  
}  
  
private LineMapper<FieldSet> payLineMapper() {  
    DefaultLineMapper<FieldSet> defaultLineMapper = new DefaultLineMapper<>();  
    
    DelimitedLineTokenizer lineTokenizer = new DelimitedLineTokenizer();  
    lineTokenizer.setDelimiter(",");  
    
    defaultLineMapper.setLineTokenizer(lineTokenizer);  
    defaultLineMapper.setFieldSetMapper(payFieldSetMapper);  
  
    return defaultLineMapper;  
}
```