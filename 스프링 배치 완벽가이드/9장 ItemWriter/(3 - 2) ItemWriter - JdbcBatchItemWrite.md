
`JdbcBatchItemWriter` 는 `JdbcTemplate` 을 살짝 감싸고 있는 래퍼이다. 네임드 파라미터`named parameter`가 데이터베이스 대량 데이터 삽입`insert`/수정`update` SQL에 사용됐는지 여부에 따라 `JdbcTEmplate.batchupdate` , `JdbcTemplate.execute` 메서드를 사용한다.

여기서 유의해야 할 점은 데이터 한 건마다 SQL 문을 한 번식 호출하는 대신 스프링은 하나의 청크에 대한 모든 SQL문을 한 번에 실행하기 위해 `PreparedStatement` 의 배치 업데이트 기능을 사용한다는 점이다.  이렇게 하면 실행 성능을 크게 향상시킬 수 있으면서도 데이터 변경 실행은 현재 트랜잭션 내에서 할 수 있다.

아래는 물음표(?) 플레이스 홀더와 `ItemPreparedStatementSetter` 를 사용해서 파일을 읽어 데이터베이스에 쓰는 단순한 예제 코드이다.

```
@Bean  
@StepScope  
public FlatFileItemReader<WriteCustomer> customerFileReaderV2(@Value("#{JobParameters['customerFile']}") Resource inputFile) {  
    return new FlatFileItemReaderBuilder<WriteCustomer>()  
            .name("customerFileReaderV2")  
            .resource(inputFile)  
            .delimited()  
            .names(new String[] {"firstName","middleInitial","lastName","address","city","state","zip"})  
            .targetType(WriteCustomer.class)  
            .build();  
  
}  
  
@Bean  
@StepScope  
public JdbcBatchItemWriter<WriteCustomer> jdbcCustomerWriter(DataSource dataSource) {  
    return new JdbcBatchItemWriterBuilder<WriteCustomer>()  
            .dataSource(dataSource)  
            .sql("INSERT INTO WRITE_CUSTOMER (first_name, middle_initial, last_name, address, city, state, zip) VALUES (?,?,?,?,?,?,?)")  
            .itemPreparedStatementSetter(new WriterCustomerItemPreparedStatementSetter())  
            .build();  
}
```

빌드한 뒤 준비된 파일을 읽으면 데이터베이스에 레코드가 쌓여있는 것을 확인할 수 있다. 위에서는 물음표 플레이스홀더를 이용해 읽은 파일을 데이터베이스에 쓰기 작업 하였다. 하지만 테이블 스키마가 변경된다면 해당 방법은 안전하지 않을수도 있다. 아래는 네임드 파라미터를 이용해 간단히 만든 `Wrtier` 이다.

```
@Bean  
@StepScope  
public JdbcBatchItemWriter<WriteCustomer> jdbcCustomerWriter(DataSource dataSource) {  
    return new JdbcBatchItemWriterBuilder<WriteCustomer>()  
            .dataSource(dataSource)  
            .sql("INSERT INTO WRITE_CUSTOMER (first_name, middle_initial, last_name, address, city, state, zip) " +  
                    "VALUES (:firstName,:middleInitial,:lastName,:address,:city,:state,:zip)")  
            .beanMapped()  
            .build();  
}
```


`beanMapped()` 를 사용하면 `ParameterSourceProvider` 구현체를 간편하게 주입받을 수 있다. 단순히 읽은 데이터를 매핑하는 것이 아닌 추가 작업이 필요하다면 해당 인터페이스를 직접 구현해야 한다.