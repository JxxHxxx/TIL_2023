
`ArrayList` 는 `List` 인터페이스를 구현하기 때문에 **데이터 저장순서가 유지되고 중복을 허용**한다.

`ArrayList` 는 `Object` 배열을 이용해서 데이터를 순차적으로 저장한다. 예를 들면, 첫 번째로 저장한 객체는 `Object` 배열의 `0`번째 위치에 저장되고 그 다음에 저장하는 객체는 `1`번째로 저장된다. 이런 식으로 계속 배열에 순서대로 저장된다.
배열에 더 이상 저장할 공간이 없으면 보다 큰 **새로운** 배열을 생성해서 기존의 배열에 저장된 내용을 새로운 배열로 **복사**한 다음 저장한다.
`ArrayList` 에서 가장 중요한 부분은 요소(`value`)를 배열에 저장한다는 것이다. 배열은 크기가 고정되어 있다는 특징이 있다. 이로 인해 요소를 추가로 저장/삭제할 때 작업이 필요하다.

`ArrayList` 를 본격적으로 파헤치기 전에 흥미를 돋궈보자.

```
@Test  
void removeElements() {  
    List<Integer> numbers1 = new ArrayList<>();  
    numbers1.addAll(IntStream.range(0, 10).boxed().toList());  
    log.info("numbers1 = {}", numbers1);  
  
    List<Integer> numbers2 = new ArrayList<>();  
    numbers2.addAll(IntStream.of(0, 1, 2, 3, 4, 100).boxed().toList());  
    log.info("numbers2 = {}", numbers2);  
  
    for (int i = 0; i < numbers1.size() - 1; i++) {  
        if (numbers2.contains(numbers1.get(i))) {  
            numbers1.remove(i);  
        }  
    }  
  
    log.info("updated numbers1 = {}", numbers1);  
  
}
```

*결과*

![[Pasted image 20230923112121.png]]

이 테스트 케이스는 내가 원하는 결과를 가져다 주지 않는다. 내가 원하는 결과는 `numbers1` 리스트에서 0, 1, 2, 3, 4 를 삭제하는 것이었다.  이러한 이유는 `ArraysList` 내부 요소를 저장하는 버퍼인 `ElementData` 에서 한 요소가 삭제될 때 마다 빈 공간을 채우기 위해 나머지 요소들이 자리를 이동하기 때문이다. 디버깅을 해보면 금방 이해할 수 있을 것이다. 


