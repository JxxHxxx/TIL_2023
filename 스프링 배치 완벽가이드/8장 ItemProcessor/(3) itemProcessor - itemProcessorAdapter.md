
스프링 배치를 사용하면 `ItemProcessorAdapter` 를 사용해 이미 개발된 다양한 서비스가 `ItemProccessor` 역할을 하도록 만들 수 있다.

이 장에서는 `ItemProcessorAdapter` 를 살펴보고, 기존에 존재하던 서비스를 어떻게 배치 잡 아이템 처리용 프로세서로 사용하는지 살펴본다.

이 기능을 살펴보기 위해 고객 이름을 대문자로 변경하는 서비스를 만들어두었다.
클래스 이름은 `UpperCaseNameService` 이다. 구현 코드는 큰 의미가 없으니 생략하도록 하겠다.

방법은 간단하다.  `ItemProcessorAdapter` 에 기존 서비스를 등록하고 리플랙션 API 를 통해 호출할 메서드를 지정하면 된다.


```
@Bean  
public ItemProcessorAdapter<CustomerV5, CustomerV5> itemProcessorAdapter(UpperCaseNameService service) {  
    ItemProcessorAdapter<CustomerV5, CustomerV5> adapter = new ItemProcessorAdapter<>();  
    adapter.setTargetObject(service);  
    adapter.setTargetMethod("upperCase");  
  
    return adapter;  
}
```


다시 잡을 실행시키면 아래와 같이 이름이 모두 대문자로 변경된 것을 확인할 수 있다.

![[Pasted image 20231212183025.png]]
