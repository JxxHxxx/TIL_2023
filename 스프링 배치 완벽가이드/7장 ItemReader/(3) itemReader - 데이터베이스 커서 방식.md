

우선 로우 매퍼를 만들고

```
public class CustomerRowMapper implements RowMapper<Customer> {

	@Override  
	public Customer mapRow(ResultSet resultSet, int rowNum) throws SQLException {  
	    Customer customer = new Customer();  
	    customer.setId(resultSet.getLong("id"));  
	    customer.setAddress(resultSet.getString("address"));  
	    customer.setCity(resultSet.getString("city"));  
	    customer.setFirstName(resultSet.getString("first_name"));  
	    customer.setLastName(resultSet.getString("last_name"));  
	    customer.setMiddleInitial(resultSet.getString("middle_initial"));  
	    customer.setState(resultSet.getString("state"));  
	    customer.setZipCode(resultSet.getString("zip_code"));  
	  
	    return customer;  
	}
}
```



JdbcCursorItemReader를 통해 ItemReader 구현

```
@Bean  
public JdbcCursorItemReader<Customer> customerItemReader(DataSource dataSource) {  
    return new JdbcCursorItemReaderBuilder<Customer>()  
            .name("customerItemReader")  
            .dataSource(dataSource)  
            .sql("select * from customer where city =?")  
            .rowMapper(new CustomerRowMapper())  
            .preparedStatementSetter(citySetter(null))  
            .build();  
}  
```


마지막으로 `ArgumentPreparedStatementSetter` 로 파라미터를 지정한다.

```
@Bean  
@StepScope  
public ArgumentPreparedStatementSetter citySetter(@Value("#{jobParameters['city']}") String city) {  
  
    return new ArgumentPreparedStatementSetter(new Object[] {city});  
}
```


참고로
`ResultSet` 은 스레드 안전`thread safe`이 보장되지 않으므로 다중 스레드 환경에서는 사용할 수 없다. 이런 단점 때문에 또 다른 선택지인 페이징을 선택하게 된다.