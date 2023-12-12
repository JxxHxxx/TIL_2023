7장에서는 다양한 유형의 입력 데이터를 읽는 방법을 살펴봤다. `ItemProcessor` 는 스프링 배치 내에서 입력 데이터를 이용해 어떤 작업을 수행하는 컴포넌트다.

##### ItemProcessor 소개
---
배치 처리의 많은 시라니오에서는 읽어온 데이터를 사용해 특정 작업을 수행해야 한다. 스프링 배치는 읽기, 처리, 쓰기 간에 고려해야 하는 문제를 잘 구분할 수 있도록 스텝을 여러 부분으로 분리했다. 이렇게 분리함으로써 다음과 같은 몇 가지 고유한 작업을 수행할 수 있다.

- **입력의 유효성 검증** : 스프링 배치 1.1x 버전에서는 `ValidatingItemReader` 클래스를 상속해서 `ItemReader` 에서 유효성 검증을 수행했다.  매번 상속해야하는 문제가 있었다.  `ItemProcessor` 가 유횽성 검증을 수행하도록 바꾸면 입력 방법에 상관없이 유효성 검증을 수행할 수 있다. 분업의 관점에서도 더 효율적이다.

- **기존 서비스의 재사용** : 어뎁터를 사용할 수 있다.

- **스크립트 실행** : 다른팀의 로직을 연결할 때 좋다. 만약 다른 팀이 스프링 기능을 사용하지 않을 때 유용하다고 한다.

- `ItemProcesor` 체인 : 동일한 트랜잭션 내에서 단일 아이템으로 여러 작업을 수행하려는 상황이 있을 수 있다. 단일 클래스 내에서 모든 로직이 수행되도록 `ItemProcessor` 를 만들 수도 있지만, 개발 로직이 프레임워크에 강하게 결합되기에 피하고 싶을 것이다. 이럴 때 스프링 배치를 사용해 각 아이템에 대해 순서대로 실행될 `ItemProcessor` 목록을 만들 수 있다.


##### ItemProcessor 특징
---

`itemProcessor` 는 단순한 전락 패턴의 인터페이스이다. 하지만 몇 가지 주의해야할 점들이 있다.

```
public interface ItemProcessor<I, O> {  
   O process(@NonNull I item) throws Exception;  
}
```


1. `null` 을 리턴한다는 것은 해당 아이템이 이후 `ItemProcessor` , `ItemWriter` 를 호출하지 않겠다는 것을 의미한다. (참고로 `reader` 에서 `null` 데이터를 반환하는 것은 더 이상 읽어들일 데이터가 없다는 것을 의미한다.)
2. `ItemProcessor` 는 멱등이어야 한다. 아이템은 내결함성(`fault tolerant`) 시나리오에서 두 번 이상 전달될 수도 있다. 즉, `ItemProcessor` 를 여러 번 실행하더라도 결과가 달라지지 않아야 한다.  내결함성이란 시스템의 어떤 부분에 고장이 발생하더라도 시스템이 설계된 대로 계속 작동하는 것을 의미한다.


##### `ValidatingItemProcessor` 맛보기
---

`ValidatingItemProcessor` 는 `ItemProcessor` 구현체로, 프로세서의 처리 전에 입력 아이템의 유효성 검증을 수행하는 스프링 배치 `Validator` 인터페이스 구현체를 사용할 수 있다. 아이템이 유효성 검증에 성공하면 이후 업무 로직 처리가 이뤄진다. 

유효성 검증에 실패하면 `ValidationException` 이 발생해 일반적인 스프링 배치 오류 처리가 절차대로 진행된다. 검증은 `JSR-303` 애노테이션을 사용할 것이다.

아래 프로젝트를 의존성에 추가하자.

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```


```
@Getter @Setter
@AllArgsConstructor  
public class CustomerV5 {  
  
    @NotNull(message = "First name is required")  
    @Pattern(regexp = "[a-zA-Z]+", message = "First name must be alphabetical")  
    private String firstName;  
  
    @Size(min = 1, max = 1)  
    @Pattern(regexp = "[a-zA-Z]+", message = "Middle initial must be alphabetical")  
    private String middleInitial;  
  
    @NotNull(message = "Last name is required")  
    @Pattern(regexp = "[a-zA-Z]+", message = "Last name must be alphabetical")  
    private String lastName;  
  
    // 생략..
}
```

`CustomerV5` 아이템에 사용할 유효성 검증 규칙을 정의했다. 이제 이 기능을 동작시키려면 스프링 배치가 각 아이템을 검증하는 매커니즘을 제공해야 한다. 이는 `ValidatingItemProcessor` 가 수행한다. 예제에서는``ValidatingItemProcessor`` 의 자손인 `BeanValidatingItemProcessor` 를 사용하도록 하겠다.

다음은 파일 예제이다.  `BeanValidatingItemProcessor` 예외 동작을 보기 위해 4행은 유효성에 어긋나는 값을 입력해두었다.

```
Aimee,C,Hoover,7341,Vel Avenue,Mobile,AL,35928  
Jonas,U,Gilbert,8852,In St.,Saint Paul,MN,57321  
Regan,M,Baxter,4851,Nec Av.,Gulfport,MS,33193  
Laura,95,Minella,8177,4th Street,Dallas,FL,04119  // middleInitial 95 유효성 X
Octavius,T,Johnson,7418,Cum Road,Houston,TX,51507
```

잡을 실행해보면 아래와 같이 `ValidationException` 예외가 발생하는 것을 볼 수 있다.

![[Pasted image 20231212154606.png]]
