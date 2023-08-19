
### 빈 문자열
길이가 0인 배열이 존재할 수 있을까? 답은 Yes이다. char형 배열도 길이가 0인 배열을 생성할 수 있고, 이 배열을 내부적으로 가지고 있는 문자열이 바로 빈 문자열이다.  `String str = "";` 과 같은 ㄴ문장이 있을 때, 참조변수 str이 참조하고 있는 String 인스턴스는 내부에 `new char[0]` 과 같이 길이가 0인 char형 배열을 저장하고 있는 것이다.

```
String whiteSpace = "";  
whiteSpace.length(); // 0
```


반면 `char` 형 변수에는 반드시 하나의 문자를 지정해야 한다.

```
char c1 = ''; // x error : empty character literal
char c2 = ' '; // 공백으로 초기화
```


일반적으로 변수를 선언할 때, 각 타입의 기본값으로 초기화 하지만 `String`은 참조형 타입의 기본값인 `null` 보다는 빈 문자열로, `char` 형은 기본값인 `\u0000` 대신 공백으로 초기화 하는 것이 보통이다.


### String 클래스의 생성자

생성자

```
String(String s) : 주어진 문자열(s)를 갖는 String인스턴스를 생성한다.

ex) String s = new String("Hello") // Hello

String(char[] value) 주어진 문자열(value)를 갖는 String인스턴스를 생성한다.

ex) char[] c = {'H', 'e', 'l', 'l','o'};
String s = new String(c); // Hello

String(StringBuffer buf) : StringBuffer 인스턴스가 갖고 있는 문자열과 같은 내용의 String 인스턴스를 생성한다.

StringBuffer sb = new StringBuffer("Hello");
String s = new String(sb); // Hello
```


### 유니코드와 보충문자

`String` 클래스 메서드 중에 매개변수의 타입이 char인 것들이 있고, int인 것들도 있다. 문자를 다루는 메서드라서 매개변수 타입이 char일 것 같은데 왜 int 일까? 바로 확장된 유니코드를 다루기 위해서이다.

유니코드는 원래 2 byte, 즉 16비트 문자체계인데, 이걸로도 모자라서 20비트로 확장하게 되었다. 그래서 하나의 문자를 char타입으로 다루지 못하고 int 타입으로 다룰 수밖에 없다. 확장에 의해 새로 추가된 문자들을 '보충 문자' 라고 한다. 매개변수가 `int` 타입이면 보충문자를 지원한다고 보면 된다.

보충 문자를 사용할 일은 거의 없으니 알고만 있으면 된다고 한다.


### String.format()

`format()`은 형식화된 문자열을 만들어내는 간단한 방법이다. `printf()`
```
String str = String.format("%d 더하기 %d는 %d입니다.", 3, 5, 3 + 5);
// str = 3 더하기 5는 8입니다.
```


### 기본형 값을 String으로 변환
숫자로 이루어진 문자열을 숫자로, 또는 그 반대로 변환하는 경우가 자주 있다. 이미 배운 것과 같이 기본형을 문자열로 변경하는 방법은 간단하다. 숫자에 빈 문자열 ""을 더해주기만 하면 된다. 이 외에도 `valueOf()`

성능은 `valueOf()` 가 더 좋지만, 빈 문자열을 더하는 방법이 간단하고 편하기 때문에 성능향상이 필요한 경우에는 `valueOf()` 가 적절하다.

```
int i = 100;
String str1 = i + ""; // 100 을 "100" 으로 변환하는 방법1
String str2 = String.valueOf(i); // 100 을 "100" 으로 변환하는 방법2
```


### String을 기본형 값으로 변환

반대로 `String`을 기본형으로 변환하는 방법도 간단하다. `valueOf()`를 쓰거나 앞서 배운 `parseInt()` 를 사용하면 된다.

```
String num = "100";  
int i1 = Integer.parseInt(num);  
int i2 = Integer.valueOf(num);
```

원래 `valueOf()` 반환 타입은 `Integer` 인데 오토박싱에 의해 `Integer`가  `int`로 자동 변환된다.

참고로 `valueOf(String s)` 는 내부에서 `parseInt(String s)` 를 호출할 뿐이다.

```
public static Integer valueOf(String s) throws NumberFormatException {  
    return Integer.valueOf(parseInt(s, 10));  // 10은 10진수를 의미
}
```

