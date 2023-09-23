
`Vector` 는 `ArrayList` 구현체가 만들어지기 이전에 존재한 구현원리와 기능적인 측면에서 동일한 객체이다. 그럼에도 기존에 작성된 소스와의 호환성을 위해 계속 남겨두고 있다. 책에서는 `ArrayList` 사용을 권장한다. 하지만 `capacity` , `size` 두 개념을 이해하는데 `Vector` 를 이용하는 것이 편리한 측면이 있기에 알아보려고 한다.


```
@Test  
void confirmVectorCapacityAndSize() {  
    Vector<Integer> v = new Vector<>(5);  
    v.add(1);  
    v.add(2);  
    v.add(3);  // 설명 (1)
    print(v);  
  
    v.trimToSize();  // 설명 (2)
    System.out.println("=== After trimToSize() ===");  
    print(v);  
  
    v.ensureCapacity(6);  // 설명 (3)
    System.out.println("=== After ensureCapacity(6) ===");  
    print(v);  
  
    v.setSize(7);  // 설명 (4)
    System.out.println("=== After setSize(7) ===");  
    print(v);  
  
    v.clear();  // 설명 (5)
    System.out.println("=== After v.clear() ===");  
    print(v);  
}  
  
public static void print(Vector v) {  
    System.out.println(v);  
    System.out.println("size : " + v.size());  
    System.out.println("capacity() : " + v.capacity());  
}
```


*결과*

```
[1, 2, 3]
size : 3
capacity() : 5
=== After trimToSize() ===
[1, 2, 3]
size : 3
capacity() : 3
=== After ensureCapacity(6) ===
[1, 2, 3]
size : 3
capacity() : 6
=== After setSize(7) ===
[1, 2, 3, null, null, null, null]
size : 7
capacity() : 12
=== After v.clear() ===
[]
size : 0
capacity() : 12
```


*용어 정리*
용량(capacity) : 버퍼에 요소(`value`)가 들어갈 수 있는 갯수
크기(size) : 버퍼에 실제로 들어가 있는 요소(`value`)의 갯수


**설명 (1)** 
`capacity` 5인 `Vector` 인스턴스에 3개의 요소를 저장한 후의 상태이다.
인스턴스는 `capacity` 5 `size` 3 이다. 인스턴스 내부 버퍼는 이러한 형태일 것이다. 
`[1, 2, 3, null, null]`

**설명(2)**
`trimToSize()` 호출로 `v` 빈 공간을 없애 `size`, `capacity` 를 같게 한다. 이 과정에서 요소를 저장하는 버퍼는 `Object[]`로 타입이 배열이다. 배열은 크기를 변경할 수 없기 때문에 새로운 배열을 생성해서 그 주소값을 변수 `v` 가 참조하게 된다. 그렇기에 기존에 생성된 `new Vector<>(5)` 더 이상 사용할 수 없으며, 후에 GC에 의해 메모리에서 제거된다.

**설명(3)**
`ensureCapacity(6)` 호출로 `capacity` 6인 배열을 새롭게 생성한다. 이때도 기존의 인스턴스가 아닌 새로운 인스턴스가 생성되고 `v` 는 새롭게 생성된 인스턴스를 참조하게 된다.

**설명(4)**
`setSize(7)` 호출로 `size`를 7이 되도록 한다. 이때 `capacity` 6 이므로 또 다시 새로운 배열 인스턴스를 만들어야 한다.  만약에 `capacity` 7보다 크다면 새로운 배열을 생성하지 않아도 된다.

**설명(5)**
`clear()` 호출로 요소를 삭제한다. `[1,2,3,null, ...]` -> `[null, null, null, null, ...]` 로 배열을 변경시킨다. 이 과정에서는 새로운 배열을 만들지 않아도 되기 때문에 `v` 의 참조 값이 변경되지 않는다.


이 과정들의 포인트는 `Vecotr` 에 어떠한 작업을 할 때 저장되어 있는 요소의 수에 따라 새로운 배열을 생성하고 기존의 배열에 값을 복사하는 과정을 거치게 된다는 점이다. 그리고 이는 효율성을 고려해야 한다면 단점이 될 수도 있다는 것이다.



