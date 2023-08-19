
기존의 다른 언어에서는 문자열을 `char` 형의 배열로 다루었으나 자바에서는 문자열을 위한 클래스를 제공한다. 이것이 바로 `String` 클래스이다.


### 변경 불가능한(immutable) 클래스

String 클래스에는 문자열을 저장하기 위해서 문자형 배열 참조변수(char[]) value를 인스턴스 변수로 정의해놓고 있다.

*책과 다른 내용*

실제 String 클래스를 살펴보면 인스턴스 변수 value 는 아래와 같이 정의되어 있다. *open JDK 17 기준*

```
/**  
 * The value is used for character storage. * * @implNote This field is trusted by the VM, and is a subject to  
 * constant folding if String instance is constant. Overwriting this * field after construction will cause problems. * * Additionally, it is marked with {@link Stable} to trust the contents  
 * of the array. No other facility in JDK provides this functionality (yet). * {@link Stable} is safe here, because value is never null.  
   
 */@Stable  
private final byte[] value;
```


다시 돌아와서 한번 생성된 String 인스턴스가 갖고 있는 문자열은 읽어 올 수만 있고, 변경할 수는 없다. 

예를 들어 아래와 같은 문자열을 변경할 경우, 인스턴스 내 문자열이 바뀌는 것이 아니라 새로운 문자열이 담긴 String 인스턴스가 생성된다.

```
String word = new String("안녕");  
System.out.println(word + " " + word.hashCode());  
word = word  + "하세요";  
System.out.println(word + " " + word.hashCode());
```


결과
```
안녕 1611021
안녕하세요 803356551
```


결과를 보면 알 수 있듯이 hashCode가 변경되었다.

다르게 말하면 아래와 같은 클래스는 인스턴스의 상태 값을 변경하더라도 인스턴스 내 값이 변경할 뿐이지, 인스턴스 자체가 새롭게 생성되지 않는다.


```
public static class MyString {  
    String value;  
  
    public MyString(String value) {  
        this.value = value;  
    }  
  
    public MyString updateValue(String value) {  
        this.value = this.value + value;  
        return this;    }  
}

public static void main(String[] args) {  
    MyString myString = new MyString("안녕");  
    MyString updateString = myString.updateValue("하세요");  
  
    System.out.println(myString.hashCode());  
    System.out.println(updateString.hashCode());  
}

결과 
321001045
321001045
```


위 코드들을 통해 인지해야 할 것은 String 클래스의 경우 문자열을 결합하는 것은 매 연산 시 마다 새로운 문자열을 가진 String 인스턴스가 생성되어 메모리 공간을 차지하게 되는 것이다. 그렇기 때문에 가능한 한 연산을 줄이는 것이 좋다고 한다.

문자열간의 결합이나 추출 등 문자열을 다루는 작업이 많이 필요한 경우에는 String 클래스 대신 StringBuffer 클래스를 사용하는 것이 좋다,


### 문자열의 비교

문자열을 만들 때는 두 가지 방법, 문자열 리터럴을 지정하는 방법과 String클래스의 생성자를 사용해서 만드는 방법이 있다.

```
String str1 = "안녕"; // 문자열 리터럴 
String str2 = new String("안녕"); // 새로운 String 인스턴스 생성
```


String 클래스의 생성자를 이용한 경우에는 new 연산자에 의해서 메모리할당이 이루어지기 때문에 항상 새로운 String인스턴스가 생성된다.

그러나 문자열 리터럴은 이미 존재하는 것을 재사용하는 것이다.
```
참고 ) 문자열 리터럴은 클래스가 메모리에 로드될 대 자동적으로 미리 생성한다.
```

다르게 말하면 문자열 리터럴을 이용하면 등가비교연사자로 비교했을 때 true 값을 얻을 수도 있다는 뜻이다.

```
String str1 = "abc";
String str2 = "abc";

str1 == str2 // true
```


### 문자열 리터럴

자바 소스파일에 포함된 모든 문자열 리터럴은 컴파일 시에 클래스 파일에 저장된다. 이 때 같은 내용의 문자열 리터럴은 **한번만** 저장된다. 문자열 리터럴도 String 인스턴스이고, 한번 생성하면 내용을 변경할 수 없으니 하나의 인스턴스를 공유하면 되기 때문이다.

클라스 파일에는 소스파일에 포함된 모든 리터럴의 목록이 있다. 해당 클래스 파일이 클래스 로더에 의해 메모리에 올라갈 때, 이 러터럴의 목록에 있는 리터럴들이 JVM내에 있는 '상수 저장소(constant pool)'에 저장된다. 이때 위 코드에 있는 `abc` 같은 문자열 리터럴이 자동적으로 생성되어 저장되는 것이다.

