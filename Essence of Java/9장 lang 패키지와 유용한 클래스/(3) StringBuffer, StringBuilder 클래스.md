
String 클래스는 인스턴스를 생성할 때 지정된 문자열을 변경할 수 없지만 `StringBuffer` 클래스는 변경이 가능하다. 내부적으로 문자열 편집을 위한 버퍼(Buffer)를 가지고 있으며, `StringBuffer` 인스턴스를 생성할 때 그 크기를 지정할 수 있다.

이 때, 편집할 문자열의 길이를 고려하여 버퍼의 길이`capacity`를 충분히 잡아주는 것이 좋다. 편집 중인 무자열이 버퍼의 길이를 넘어서게 되면 버퍼의 길이를 늘려주는 작업이 추가로 수행되어야하기 때문에 작업효율이 떨어진다.

참고로 StringBuffer 클래스 생성시 `capacity` 를 지정하지 않으면 16으로 초기화 된다.

```
public StringBuffer() {
	super(16); // 인자로 capacity를 받음
}
```


실제로 아래 코드를 디버깅해보면 버퍼가 늘어남을 볼 수 있다.

```
StringBuffer sb = new StringBuffer();  
sb.append("안녕안녕안녕안녕안녕안녕안녕");  
sb.append("하세요하세요하세요하세요하세요하세요하세요");  
```

![[Pasted image 20230819214528.png]]

![[Pasted image 20230819214538.png]]

배열의 길이는 변경될 수 없기때문에 새로운 길이의 배열을 생성한 후에 이전 배열의 값을 복사해서 이러한 작업을 지속한다.


### StringBulider

`StringBuffer`는 멀티쓰레드에 안전하도록 동기화되어 있다. 하지만 동기화로 인해 `StringBuffer`의 성능은 하락한다. 따라서 멀티쓰레드 환경이 아니라면 StringBuffer의 동기화는 불필요하게 성능만 떨어트리게 된다.

그래서 `StringBuffer` 에서 쓰레드의 동기화만 뺀 `StringBuilder` 가 새로 추가되었다. `StringBuffer` 도 충분히 성능이 좋기 때문에 성능향상이 반드시 필요한 경우를 제외하고는 기존에 작성한 코드에서 `StringBuffer` 를 `StringBuilder` 로 굳이 바꿀 필요는 없다고 한다.


