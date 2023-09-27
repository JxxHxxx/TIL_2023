

### Environment
---

`Environment`  를 이용하면 외부 설정값을 `getProperty` 메서드를 이용해서 일관되게 읽을 수 있다.


*application.properties*

```
my.datasource.url=local.db.com 
my.datasource.username=local_user 
my.datasource.password=local_pw 
my.datasource.etc.max-connection=1 
my.datasource.etc.timeout=3500ms 
my.datasource.etc.options=CACHE,ADMIN
```


*설정 코드*

```
@Slf4j
@Configuration 
public class MyDataSourceEnvConfig { 
	private final Environment env; 

	public MyDataSourceEnvConfig(Environment env) { 
		this.env = env; 
		} 
		
	@Bean 
	public MyDataSource myDataSource() { 
		String url = env.getProperty("my.datasource.url"); 
		String username = env.getProperty("my.datasource.username"); 
		String password = env.getProperty("my.datasource.password"); 
		int maxConnection = env.getProperty("my.datasource.etc.max-connection", Integer.class); 
		Duration timeout = env.getProperty("my.datasource.etc.timeout", Duration.class); 
		List options = env.getProperty("my.datasource.etc.options", List.class);
	 
		return new MyDataSource(url, username, password, maxConnection, timeout, options); 
		} 
	}
```


앞서 말했듯 `Environment` 를 사용하면 외부 설정의 종류(`OS` , `VM options` ,`커맨드 라인 인수`)와 관계없이 일관성 있게 외부 설정을 조회할 수 있다.



### @Value
---

`@Value` 를 사용하면 외부 설정값을 편리하게 주입받을 수 있다. 참고로 `@Value` 도 내부에서는 `Environment` 를 사용한다.



```
@Value("${my.datasource.url}") 
private String url; 
@Value("${my.datasource.username}") 
private String username;
```

- `@Value` 에 `${}` 를 사용해서 외부 설정의 키 값을 주면 원하는 값을 주입 받을 수 있다.


###### @Value 기본값

`@Value` 를 이용하면 기본 값을 지정할 수 도 있다.

`@Value("${my.datasource.etc.max-connection:1}")` 이렇게  콜론(`:`) 으로 구분자를 넣고 값을 넣으면 된다. 이 경우 외부 설정 파일(`properties` , `yml`)에 키가 없을 경우 `1` 로  `my.datasource.etc.max-connection` 에 대한 값이 지정된다.