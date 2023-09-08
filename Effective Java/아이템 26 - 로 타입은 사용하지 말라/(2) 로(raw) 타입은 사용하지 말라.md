
클래스와 인터페이스 선언에 타입 매개변수가 쓰이면, 이를 제네릭 클래스 혹은 제네릭 인터페이스라 한다. 그리고 이를 통틀어 제네릭 타입이라 한다.  
  
```  
// 제네릭 타입 예시  
public class SimpleResponse<T> {  
private final T response;  
//...   
}  
```  
  
  
위 코드를 예로 들어 `SimpleResponse<String>` 를 선언했다고 하자. `String` 이 정규 타입 매개변수 `T` 에 해당하는 **실제 타입 매개변수**이다.  
  
마지막으로, 제네릭 타입을 하나 정의하면 그에 딸린 **로 타입**`raw type`도 함께 정의된다. 로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다. 예를 들어 `SimpleResponse<T>` 의 raw 타입은 `SimpleResponse` 이다.


### 로 타입은 왜 안티 패턴일까?

----

로 타입에 타입 매개변수를 지정하지 않으면 `Object` 가 들어가게 된다. 여러 가지 타입이 들어갈 가능성이 생기게 된다. 이 경우에는 캐스팅 오류, 컬렉션에서 가져온 객체를 처리할 때 많은 부분을 고려하며 짜야되기 때문에 코드의 복잡도가 증가할 수 밖에 없다.


*타협 1*
그래서 책에서는 차라리 `SimpleResponse` 보다는 `SimpleResponse<Object>` 를 권장한다. 이럴 경우에는 타입 안전성은 여전히 떨어지지만 표현력은 유지할 수 있다.
여기서 말하는 표현력이란 `SimpleResponse<Object>` 를 선언함으로서 해당 인스턴스에는 모든 객체가 들어갈 수 있음을 명시하는 것을 의미한다.

*타협 2*
책에서 말하는 해결책은 아니다. 무조건적으로 제네릭을 선언하는게 안전성 측면에서는 나을 수 밖에 없다. 하지만 과도한 제네릭 사용은 가독성을 낮춘다.

```
@GetMapping("/reviews/{review-id}")
public ResponseEntity<DataResponse<ReviewResult<ReviewServiceResponse>>> findReview(..) {
	//.. 컨트롤러 로직 처리
}
```

이해하기 쉽지 않을 것이다. 차라리 아래처럼 작성하는 것이 어떨까?


```
@GetMapping("/reviews/{review-id}")
public ResponseEntity<DataResponse> findReview() {
}
```

오히려 나는 이게 좋다고 본다. 컨트롤러는 웹 개발에서 시작이자 끝이다.  (필터, 인터셉터 등은 고려하지 말자) 제네릭 타입이 마구잡이로 들어가더라도 `return` 되고 끝일 뿐이지 다른 계층에서 사용하지 않는다. 

타입 안전성이 상대적으로 중요하지 않은 부분에서는 과감하게 포기하고 가독성을 높이는게 좋다고 생각한다.