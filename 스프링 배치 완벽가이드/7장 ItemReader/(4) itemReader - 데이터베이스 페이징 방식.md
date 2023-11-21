
페이징 기법을 사용할 때도 커서 기법과 마찬가지로 잡이 처리할 아이템은 여전히 한 건씩 처리된다는 점에 주목하기를 바란다. 레코드 처리 자체에는 차이가 없다.

커서 기법과 차이가 있는 부분은 데이터베이스에서 가져오는 방법이다.

페이징 기법은 각 페이지마다 새로운 쿼리를 실행한 뒤 쿼리 결과를 한 번에 메모리를 적재한다. 그래서 페이징 기법을 사용하려면 페이지 크기(반환될 레코드 개수)와 페이지 번호(현재 처리중인 페이지)를 가지고 쿼리를 할 수 있어야 한다.

예를 들어 전체 레코드 개수 10,000개이고 한 페이지의 크기가 100 레코드라고 하자. 이 때 20번째(2,0001부터 2,100 레코드) 페이지를 요청한다는 것을 조건으로 지정할 수 있어야 한다. 이러한 기능은 `PagingQueryProvider` 인터페이스를 통해 이를 구현할 수 있다.

아쉬운 점은 각 데이터베이스마다 개별적인 페이징 구현체를 제공한다. 그래서 2가지 선택지가 있다.

1. 사용하는 데이터베이스 전용 `PagingQueryProvider` 구현체를 구성한다. 스프링 배치에서는 `DB2` , `H2` ,`MySQL` , `Oracle` , `Postgres` ,`Mssql` , `Sybase` , `Derby` `Hsql`  구현체를 제공한다.
2.  `reader` 가 `SqlPagingQueryProviderFactoryBean` 을 사용하도록 구성한다. 이 팩토리는 사용하는 데이터베이스가 어떤 것인지 감지할 수 있다.

`JdbcPagingItemReader`를 구성하려면 네 가지 의존성이 필요하다. 
- `DataSource` 
- `PagingQueryProvider` 구현체
- 직접 개발한 `RowMapper` 구현체
- 페이지의 크기


아래는`SqlPagingQueryProviderFactoryBean` 를 사용해서 페이징 `Reader`를 구현한 예시다.


```
@Bean  
@StepScope  
public JdbcPagingItemReader<Customer> customerItemReader(
	DataSource dataSource,  
    PagingQueryProvider queryProvider,  
    @Value("#{jobParameters['city']}") String city) {  
    HashMap<String, Object> parameterValues = new HashMap<>(1);  
    parameterValues.put("city", city);  
  
    return new JdbcPagingItemReaderBuilder<Customer>()  
            .name("customerItemReader")  
            .dataSource(dataSource)  
            .queryProvider(queryProvider)  
            .parameterValues(parameterValues)  
            .pageSize(10)  
            .rowMapper(new CustomerRowMapper())  
            .build();  
}  
  
@Bean  
public PagingQueryProvider pagingQueryProvider(DataSource dataSource) throws Exception {  
    SqlPagingQueryProviderFactoryBean factoryBean = new SqlPagingQueryProviderFactoryBean();  
    factoryBean.setSelectClause("select *");  
    factoryBean.setFromClause("from Customer");  
    factoryBean.setWhereClause("where city = :city");  
    factoryBean.setSortKey("last_name");  
    factoryBean.setDataSource(dataSource);  
    return factoryBean.getObject();  
}
```


Step에는 위에서 구현한 `reader` 를 넣어주면 된다.

```
@Bean  
public Step copyFileStep() throws Exception {  
	return this.stepBuilderFactory.get("copyFileStep")  
		.<Customer, Customer> chunk(10)  
		.reader(customerItemReader(dataSource, pagingQueryProvider(dataSource), null))  
		.writer(itemWriter())  
		.build();  
}
```


