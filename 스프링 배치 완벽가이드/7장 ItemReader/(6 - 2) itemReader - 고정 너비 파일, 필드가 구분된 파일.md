
이 장에서는 여러 파일 형식을 `reader` 가 처리하는 방식을 알아본다.


##### 고정 너비 파일
---
아래는 고정 너비 파일의 포맷이다. 구분자가 존재하지 않는 것이 특징이다.

```
Aimee      CHoover    7341Vel Avenue          Mobile          AL35928  
Jonas      UGilbert   8852In St.              Saint Paul      MN57321  
Regan      MBaxter    4851Nec Av.             Gulfport        MS33193  
Octavius   TJohnson   7418Cum Road            Houston         TX51507
```


앞서 `LineTokenizer` 를 통해 줄(`Line`)을 파싱해서 `Fieldset` 을 만든다고 했다. 예시에서는 `fixedLength()` 를 통해 `FixedLengthTokenizer` 를 등록한다.


```
@Bean  
@StepScope  
public FlatFileItemReader<Customer> customerItemReader(@Value("#{jobParameters['customerFile']}") Resource inputFile) {  
    return new FlatFileItemReaderBuilder<Customer>()  
            .name("customerItemReader")  
            .resource(inputFile)  
            .fixedLength()  // FixedLengthTokenizer 등록
            .columns(new Range[]{new Range(1, 11), new Range(12, 12), new Range(13, 22), new Range(23, 26), new Range(27, 46)  , new Range(47, 62), new Range(63, 64), new Range(65, 69)})  
            .names(new String[]{"firstName", "middleInitial", "lastName", "addressNumber", "street", "city", "State", "zipCode"})  
            .targetType(Customer.class)  
            .strict(false)  
            .build();  
}
```



##### 필드가 구분자로 구분된 파일
---
구분자로 구분된 파일(`delimited file`)은 파일의 포맷이 어떠한지 나타내는 메타데이터가 파일 내에 소량 제공된다. 하지만 각 필드의 정의를 알 수는 없다. 처리 방식은 고정 너비 레코드와 거의 유사하다.

먼저 `LineTokenizer` -> `FieldSet` 로 만들고 `FieldSetMapper` 를 사용해 도메인 객체로 매핑한다.


```
@Bean  
@StepScope  
public FlatFileItemReader<Customer> customerItemReader(@Value("#{jobParameters['customerFile']}") Resource inputFile) {  
    return new FlatFileItemReaderBuilder<Customer>()  
            .name("customerItemReader")  
            .resource(inputFile)  
            .delimited()
            .names(new String[]{"firstName", "middleInitial", "lastName", "addressNumber", "street", "city", "State", "zipCode"})
            .targetType(Customer.class)  
            .strict(false)  
            .build();  
}
```


`lineTokenizer` 를 구성하는 아래 부분은 다음과 같이 직접 구현할 수도 있다.

```
.delimited()
.names(new String[]{"firstName", "middleInitial", "lastName", "addressNumber", "street", "city", "State", "zipCode"})
```


```
@Component  
public class CustomerFileLineTokenizer implements LineTokenizer {  
  
    private final String delimiter = ",";  
    private String[] names = new String[]{"firstName", "middleInitial", "lastName", "addressNumber", "street", "city", "State", "zipCode"};  
  
    @Override  
    public FieldSet tokenize(String line) {  
        DelimitedLineTokenizer lineTokenizer = new DelimitedLineTokenizer();  
        lineTokenizer.setDelimiter(delimiter);  
        lineTokenizer.setNames(names);  
  
        return lineTokenizer.tokenize(line);  
    }  
}
```
