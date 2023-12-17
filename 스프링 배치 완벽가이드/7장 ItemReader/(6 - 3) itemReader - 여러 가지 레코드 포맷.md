6 - 1,2 에서는 고객 레코드가 담긴 파일을 처리하는 방법을 살펴봤다. 파일 내 각 레코드는 완벽히 동일한 포맷이었다. 그러나 고객 정보뿐만 아니라 고객의 거래 정보까지 담긴 파일을 받았을 때는 어떻게 해야 할까?

커스텀 `LineTokenizer` 하나를 새로 구현하면 된다. 하지만 이런 접근법에는 두 가지 문제점이 있다.

1. 복잡도 : 파일 내에 세 가지, 네 가지, 다섯 가지, 아니 그 이상의 레코드 포맷이 존재한다고 해보자. 또 각 레코드 포맷마다 무수히 많은 필드가 포함돼 있다고 해보자. 곧 `Lineokenizer` 클래스 하나로는 감당할 수 없게 된다.
2. 관심사 분리 : `LineTokenizer` 의 목적은 레코드를 파싱하는 것 그 이상, 이하도 아니다. 레코드 파싱을 넘어 어떤 레코드 유형인지를 판별하는데 사용해서는 안 된다.

##### `PartternMatchingCompositeLineMapper`
---
스프링 배치는 이런 점을 감안해 별도의 `LineMapper` 구현체 `PartternMatchingCompositeLineMapper` 를 제공한다. 앞서 우리는 `DefaultLineMapper` 구현체를 사용했다. `DefaultLineMapper` 는 `LineTokenzier` 하나 `FileSetMapper` 하나를 사용해 매핑 기능을 제공한다.

반면 `PartternMatchingCompositeLineMapper` 를 사용하면 여러 `LineTokenizer` 로 구성된 `Map` 을 선언할 수 있으며 각 `LineTokenizer` 가 필요로 하는 여러 `FieldSetMapper` 로 구성된 `Map` 을 선언할 수 있다. 각 맵의 키는 레코드의 패턴이다. `LineMapper` 는 이 패턴을 이용해서 각 레코드를 어떤 `LineTokenizer` 로 파싱할지 식별한다.


예제를 살펴보자.

먼저 우리가 처리해야할 파일이다. 각 Line은 그 의미가 `CUST` , `TRANS` 로 구분되어 있다. 

```
CUST,Warren,Q,Darrow,8272,4th Street,New York,IL,76091  
TRANS,1165965,2011-01-22 00:13:29,51.43  
CUST,Ann,V,Gates,9247,Infinite Loop Drive,Hollywood,NE,37612  
CUST,Erica,I,Jobs,8875,Farnam Street,Aurora,IL,36314  
TRANS,8116369,2011-01-21 20:40:52,-14.83  
TRANS,8116369,2011-01-21 15:50:17,-45.45  
TRANS,8116369,2011-01-21 16:52:46,-74.6  
TRANS,8116369,2011-01-22 13:51:05,48.55  
TRANS,8116369,2011-01-22 16:51:59,98.53
```


이후 아래와 같이 `PatternMatchingCompositeLineMapper` 를 구현하면 된다. 주목할만한 점은 `LineTokenizer` , `FieldSetMapper` 를 담는 Map 의 키다. `CUST*` 는 줄(`line`)이 CUST로 시작하면 `customerFileLineTokenizer` , `customerFieldSetMapper` 로 전달하라는 의미이다.

```
@Bean  
public PatternMatchingCompositeLineMapper lineMapper() {  
    HashMap<String, LineTokenizer> lineTokenizers = new HashMap<>(2);  
  
    lineTokenizers.put("CUST*", customerFileLineTokenizer);  
    lineTokenizers.put("TRANS*", transactionLineTokenizer);  
  
    HashMap<String, FieldSetMapper> fieldSetMappers = new HashMap<>(2);  
  
    fieldSetMappers.put("CUST*", customerFieldSetMapper);  
    fieldSetMappers.put("TRANS*", transactionFieldSetMapper);  
  
    PatternMatchingCompositeLineMapper lineMappers = new PatternMatchingCompositeLineMapper<>();  
    lineMappers.setTokenizers(lineTokenizers);  
    lineMappers.setFieldSetMappers(fieldSetMappers);  
  
    return lineMappers;  
}
```

