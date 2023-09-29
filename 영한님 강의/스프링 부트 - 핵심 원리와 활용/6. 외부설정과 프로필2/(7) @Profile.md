각 환경마다 다른 설정 값이 아니라 아예 다른 `@Bean` 을 등록해야 한다면 어떻게 해야할까? 이 때 `@Profile` 애노테이션을 사용할 수 있다.


```
@Configuration  
public class PayConfig {  
  
    @Bean  
    @Profile("default")  
    public LocalPayClient localPayClient() {  
        log.info("LocalPayClient 빈 등록");  
        return new LocalPayClient();  
    }  
  
    @Bean  
    @Profile("prod")  
    public ProdPayClient prodPayClient() {  
        log.info("ProdPayClient 빈 등록");  
        return new ProdPayClient();  
    }  
}
```


이제  `OS`변수, `VM options` 등에 `spring.profiles.active={profile 명}` 을 넣어주면 해당 `@Bean` 이 등록된다.