
`FlatFileItemWriter` 는 쓰기 데이터의 노출을 제한해 롤백이 가능한 커밋 전 마지막 순간까지 출력 데이터의 저장을 지연시킨다. 즉, 쓰기 외 모든처리 작업을 완료한 뒤 라이터로 데이터를 실제로 디스크에 기록하기 직전에 트랜잭션을 커밋한다고 한다.

그 이유는 파일의 경우 데이터가 디스크로 플러시되면 롤백할 수 없기 때문이다. 

다음은 파일을 읽어(`read`) 새로운 포멧으로 파일을 쓰는(`write`) 잡 코드의 일부이다.

```
@Bean  
@StepScope  
public FlatFileItemReader<WriteCustomer> customerFileReader(@Value("#{jobParameters['customerFile']}") Resource inputFile) {  
    return new FlatFileItemReaderBuilder<WriteCustomer>()  
            .name("customerFileReader")  
            .resource(inputFile)  
            .delimited()  
            .names(new String[] {"firstName","middleInitial","lastName","address","city","state","zip"})  
            .targetType(WriteCustomer.class)  
            .build();  
}  
  
@Bean  
@StepScope  
public FlatFileItemWriter<WriteCustomer> customerItemWriter(@Value("#{jobParameters['outputFile']}") Resource outputFile) {  
  
    return new FlatFileItemWriterBuilder<WriteCustomer>()  
            .name("customerItemWriter")  
            .resource(outputFile)  
            .formatted()   
            .format("%s %s lives at %s %s in %s, %s.")  
            .names(new String[]{"firstName", "lastName","address","city","state","zip"})  
            .build();  
}
```


빌드 시에 주의해야 할 점은 리소스를 쓸 때(`write`)는 file URL 스키마를 사용해야 한다.

예를 들어 아래와 같이 `file:` 을 시작으로 리소스를 쓰고 저장할 경로를 지정해야 한다.
```
outputFile=file:/Users/JH/Desktop/hello_spring_batch/formattedCustomers.txt
```

![[Pasted image 20231217133457.png]]
