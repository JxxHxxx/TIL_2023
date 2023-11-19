
`JsonItemReader` 를 통해 `JSON` 포멧 데이터를 읽을 수 있다.  `JsonItemReader` 구성을 위해서는
`JsonItemReaderBuilder` 를 사용한다. 필요한 것은 총 3가지이다.

- 배치의 이름 : `name()` 
- `json` -> 객체로 파싱 : `jsonObjectReader()`
- 읽을 리소스 : `resource()`


아래는 `JsonItemReader` 를 실제로 구현한 예시 코드이다.

```
@Bean  
@StepScope  
public JsonItemReader<Customer> customerFileReader(@Value("#{jobParameters['customerFile']}")Resource inputFile) {  
    log.info("resource class = {}", inputFile.getClass());  
  
    ObjectMapper objectMapper = new ObjectMapper();  
    objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));  
  
    JacksonJsonObjectReader<Customer> jsonObjectReader = new JacksonJsonObjectReader<>(Customer.class);  
    jsonObjectReader.setMapper(objectMapper);  
  
    return new JsonItemReaderBuilder<Customer>()  
            .name("customerFileReader")  
            .jsonObjectReader(jsonObjectReader)  
            .resource(inputFile)  
            .build();  
}
```


이외에 배치 실행을 위해 필요한 소스 코드이다.

```
@Bean  
public Job copyFileJob() {  
    return this.jobBuilderFactory.get("job")  
            .start(copyFileStep())  
            .incrementer(new RunIdIncrementer())  
            .build();  
}  
  
@Bean  
public Step copyFileStep() {  
    return this.stepBuilderFactory.get("copyFileStep")  
            .<Customer, Customer> chunk(10)  
            .reader(customerFileReader(null))  
            .writer(itemWriter())  
            .build();  
}  

@Bean  
public ItemWriter<Customer> itemWriter() {  
    return (items -> items.forEach(System.out::println));  
}  
```


마지막으로 읽어들일 `json` 파일이다.

위치 : `main/resources/customer.json`

```
[  
  {  
    "firstName" : "Laura",  
    "middleInitial" : "0",  
    "lastName" : "Minella",  
    "address" : "2039 Wall Street",  
    "city" : "Omaha",  
    "state" : "IL",  
    "zipCode" : "35446",  
    "transactions" : [  
      {  
        "accountNumber" : 829433,  
        "transactionDate" : "2010-10-14 05:49:58",  
        "amount" : 26.08  
      }  
    ]  
  },  
  {  
    "firstName" : "Michael",  
    "middleInitial" : "T",  
    "lastName" : "Buffett",  
    "address" : "8192 Wall Street",  
    "city" : "Omaha",  
    "state" : "NE",  
    "zipCode" : "25372",  
    "transactions" : [  
      {  
        "accountNumber" : 8179238,  
        "transactionDate" : "2010-10-27 05:56:59",  
        "amount" : -91.76  
      },  
      {  
        "accountNumber" : 8179238,  
        "transactionDate" : "2010-10-06 21:51:05",  
        "amount" : -25.99  
      }  
    ]  
  }  
]
```



다시 빌드를 한 뒤 아래와 같이 `customerFile` 파라미터에 리소스 위치를 지정해주면 된다.

```
java -jar {build filename}.jar customerFile=/customer.json
```