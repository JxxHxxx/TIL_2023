
이 장에서는 기존 스프링 서비스를 호출해서 `ItemReader` 에 데이터를 공급하는 방법을 알아본다.

앞서 우리는 4장에서 스프링 배치가 태스크릿에서 별도의 기능을 사용할 수 있게 하는 어댑터 3가지를 배웠다.

- `CallableTaskletAdapter`
- `MethodInvokingTaskletAdapter`
- `SystemCommandTasklet`

이 세개의 어댑터는 다른 엘리먼트를 래핑해서 스프링 배치가 해당 엘리먼트와 통신할 수 있게 하는 데 사용된다. 스프링 배치에서 기존 서비스를 사용할 때도 같은 패턴을 사용한다.

입력 데이터를 읽을 때는 `ItemReaderAdapter` 를 사용한다. `ItemReaderAdpater` 사용 시 다음 2가지를 염두해야 한다.

1. 매번 호출할 때마다 반환되는 객체는 `ItemReader` 가 반환하는 객체다.
2. 입력 데이터를 모두 처리하면 `null` 반환

##### 예시
---
스프링 프로젝트를 공부하다 보면 `adapter` 가 종종 등장하다. 유연하게 결합하는 포인트를 제공한다고 보면 된다.

이 어뎁터를 어디에 활용할 수 있을지 예시를 들어보도록 하겠다. 우리가 이미 배치 프로젝트가 아닌 곳에서 아이템을 읽는 서비스 컴포넌트를 만들었다고 해보자. 이럴 때 간단하게 어뎁터로 연결해서 사용할 수 있다.

*예시 코드*

배치 처리가 아닌 api 모듈을 개발하면서 `customerService` 클래스를 구현했다고 해보자. 배치 기능을 만들 때 처음부터 다시 만드는건 비효율적일 수도 있다. 이때 `ItemReaderAdapter` 의 `reflection` 을 통해 간단하게 연결시켜 읽어들일 데이터를 지정할 수 있다.

```
public ItemReaderAdapter<T> customerItemReader(CustomerService customerService) {  
    ItemReaderAdapter<T> adapter = new ItemReaderAdapter<>();  
  
    adapter.setTargetObject(customerService);  
    adapter.setTargetMethod("getCustomer");  
  
    return adapter;  
}
```

