
처리 작업(`Process`)을 단일 `ItemProcessor` 로 모두 몰아두는 것이 적절하지 않을 수 있다. 이 장에서는 `ItemProcessor` 를 체인처럼 연결할 수 있게 해주는 컴포지션(`Composition`) 을 정리하도록 하겠다.


##### `CompositeItemProcessor`
---

사용법은 간단하다. 앞서 `CompositeItemReader` 사용법과 광장히 유사하다.

```
    @Bean  
    public CompositeItemProcessor<CustomerV5, CustomerV5> compositeItemProcessor() {  
        CompositeItemProcessor<CustomerV5, CustomerV5> compositeItemProcessor = new CompositeItemProcessor<>();  
        List<? extends ItemProcessor<CustomerV5, CustomerV5>> itemProcessors = List.of(
        itemProcessorAdapter(null), // (1) 실행
        validatingItemProcessor() // (2) 실행
        );  
  
        compositeItemProcessor.setDelegates(itemProcessors);  
  
        return compositeItemProcessor;  
    }
```

`CompositeItemProcessor` 의 구현체를 보면 등록된 `ItemProcessor` 의 순서를 보장한다. 따라서 순서를 고려해서 프로세스를 등록하자.
