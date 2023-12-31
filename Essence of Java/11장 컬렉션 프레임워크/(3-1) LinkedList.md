
배열은 데이터를 읽어오는데 걸리는 시간(접근시간, access time)이 가장 빠르다는 정점이 존재한다.

하지만 단점도 존재한다.

```
1. 크기를 변경할 수 없다.
- 크기를 변경할 수 없으므로 새로운 배열을 생성해서 데이터를 복사해야한다.
- 실행속도를 향상시키기 위해서는 충분히 큰 크기의 배열을 생성해야 하므로 메모리가 낭비된다

2. 비순차적인 데이터의 추가/삭제에 시간이 많이 걸린다.
- 차례대로 데이터를 추가하고 마지막에서부터 데이터를 삭제하는 것은 빠르지만
- 배열의 중간에 데이터를 추가하려면, 빈자리를 만들기 위해 다른 데이터들을 복사해서 이동해야 한다.
```

이러한 배열의 단점을 보완하기 위해서 `LinkedList`라는 자료구조가 고안되었다. 배열은 모든 데이터가 연속적으로 존재하지만 `LinkedList` 는 불연속적으로 존재하는 데이터를 서로 연결(link)한 형태로 구성되어 있다. 아래 그름을 보면 `ArrayList` , `LinkedList` 차이를 알 수 있다.

*생활코딩 - list 강의 내 이미지*

![[Pasted image 20230924121205.png]](Pasted%20image%2020230924121205.png)


`LinkedList` 각 요소(`node`)들은 자신과 연결된 다음(`next`) 요소 참조(주소값)와 데이터로 구성되어 있다.

```
Class Node {
	Node next; // 다음 요소의 주소를 저장
	Object ojb; // 데이터 저장
}
```


*LinkedList에서 데이터 삭제*

삭제하고자 하는 요소의 이전(`previous`)요소가 삭제하고자 하는 요소의 다음 요소를 참조하도록 변경하기만 하면 된다. **단 하나의 참조만 변경하면 삭제가 이루어지는 것이다.** 배열처럼 데이터를 이동하기 위해 복사하는 과정이 없기 때문에 처리속도가 매우 빠르다.

*LinkedList에서 데이터 추가*

삭제와 유사하다. 

1. 새로운 요소를 만들고 추가 하고자 하는 위치의 이전(`previous`) 요소의 `next` 필드가 새롭게 추가하고자 하는 요소를 바라보게 한다.
2.  새롭게 추가된 요소는 이전 요소의 `next` 필드가 기존에 바라보고 있던 참조를 바라보게 하면 된다.

*더블 링크드 리스트*

링크드 리스트 내 요소(`Node`)는 `next` 필드와 더불어 `prev` 를 가지고 있다. 

![[Pasted image 20230924122542.png]]

이는 이전 요소에 대한 접근을 쉽게 하기 위해서이다. 여기서 한걸음 더 나아간 것이 `더블 서큘러 링크드 리스트`이다.  더블 링크드 리스트에서 첫 번째 요소와 마지막 요소를 연결시킨 것이다.


##### 결론

---

|컬렉션|읽기(접근시간)|추가/삭제|비고|
|---|---|---|---|
|ArrayList|빠르다|느리다|순차적인 추가삭제는 더 빠름
|LinkedList|느리다|빠르다|데이터가 많을수록 접근성이 떨어짐|


