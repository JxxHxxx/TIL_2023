
앞서 `@ConfigurationProperties` 를 통해 타입 안정성을 더할 수 있었다. 이제 문제는 설정 값들의 범위와 같은 유효성이다.

*build.gradle 추가*

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```




```
@Getter  
public static class Etc {  
    @Min(1) @Max(999)  
    private int maxConnection;  
    @DurationMin(seconds = 1)  
    @DurationMax(seconds = 60)  
    private Duration timeout;  
    private List<String> options;  
  
    public Etc(int maxConnection, Duration timeout, List<String> options) {  
        this.maxConnection = maxConnection;  
        this.timeout = timeout;  
        this.options = options;  
    }  
}
```


*검증 시, 유효성이 어긋날 때*

![[Pasted image 20230928155356.png]](Pasted%20image%2020230928155356.png)
