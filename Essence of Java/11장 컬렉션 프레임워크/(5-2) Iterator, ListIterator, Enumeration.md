
`Iterator`, `ListIterator`, `Enumeration` 모두 컬렉션에 저장된 요소를 접귾는데 사용되는 인터페이스다. `Enumeration` 은  `Iterator` 구버전,  `ListIterator` 은 `Iterator` 기능을 향상 시킨 것이다.


##### Iterator

`Iterator` 는 컬렉션(`Collections`) 인터페이스에 저장된 요소들을 읽어오는 표준이다.  그리고 `Collection` 자손인 `List` , `Set` 에도 포함되어 있다. 다음과 같이 구현체에서도 호출 할 수 있다.

```
@Test  
void execute_iter() {  
    List<String> words = new ArrayList<>();  
    words.addAll(List.of("hello world", "my name is jxx", "the end"));  
  
    Iterator<String> iter = words.iterator();   // 호출
  
    while (iter.hasNext()) {  
        System.out.println(iter.next());  
    }  
}
```



##### Iterator 와 Map

`Map` 인터페이스를 구현한 컬렉션 클래스는 `key` , `value` 쌍으로 저장하고 있기 때문에 `iterator()` 를 직접 호출할 수 없습니다. 대신 `keySet()` , `entrySet()` 과 같은 메서드를 통해서 `key` , `value` 값을 각각 따로 `Set` 형태로 얻어 온 후에 다시 `iterator()`를 호출해야 `Iterator` 를 얻을 수 있습니다.

```
@Test  
void execute_iter_in_map() {  
    Map<String, Object> map = new HashMap<>();  
    Iterator<Map.Entry<String, Object>> iter = map.entrySet().iterator();  
}
```

