

우리는 VM options `-Dmemory=on` 설정을 해두었다.

### @ConditionalOnProperty

앞서 `Condition` 인터페이스를 구현한 클래스와 `@Condition` 을 사용해서 특정 환경에서만 빈을 읽도록 했다. `@ConditionalOnProperty` 을 이용하면 더 쉽게 앞서 만든 환경을 구성할 수 있다.

```
@Configuration    
@ConditionalOnProperty(name = "memory", havingValue = "on")  
public class MemoryConfig {  
  
    @Bean  
    public MemoryController memoryController() {  
        return new MemoryController(memoryFinder());  
    }  
  
    @Bean  
    public MemoryFinder memoryFinder() {  
        return new MemoryFinder();  
    }  
}
```


이외에도 [@ConditionalXxx 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration.condition-annotations)를 참고하면 다양한 `@Conditional` 애노테이션을 사용할 수 있다.