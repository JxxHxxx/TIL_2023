
이 장에서는 데이터를 갱신하는 방법을 정리하겠습니다. 테이블의 셀에 저장되어 있는 값을 갱신하려면 UPDARE 명령을 사용합니다.

## 목차
1. [UPDATE로 데이터 갱신하기]
2. [SET 구의 실행 순서]

## UPDATE로 데이터 갱신하기

`UPDATE {table_name} SET {column_name} = {value} WHERE {condition}`

주의해야할 점

1. 값(value)은 상수로 표기해야 합니다.
2. 해당 열에 맞는 자료형으로 값을 지정해야 합니다.
3. 갱신해야할 열이 복수 개인 경우 `SET {column_name} = {value1}, {column_name} = {value2}` 이런 식으로 콤마로 구분자를 설정합니다.
4. SET 구의 = 은 대입 연산자입니다.


## SET 구의 실행 순서

아래  UPDATE 명령은 콤마(,)로 구분된 갱신 식의 순서가 서로 다릅니다.  2가지 명령의 결과는 동일할까요?

```
1. UPDATE sample1 SET no = no + 1, val1 = no;  
2. UPDATE sample1 SET val1 = no, no = no + 1;
```

결과는 **데이터베이스 마다 다르다고 합니다.**

예를 들어 MySQL에서는 서로 다른 결괏값이 나오지만 Oracle에서는 어느 명령을 실행해도 결과는 같습니다.

먼저 초기 테이블은 아래와 같습니다.

![[Pasted image 20230810133213.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/4%EC%9E%A5%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%94%EA%B0%80%2C%20%EC%82%AD%EC%A0%9C%2C%20%EA%B0%B1%EC%8B%A0/Pasted%20image%2020230810133213.png)

먼저 1번 UPDATE 명령의 결과를 보겠습니다.
`UPDATE sample1 SET no = no + 1, val1 = no;`

![[Pasted image 20230810133244.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/4%EC%9E%A5%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%94%EA%B0%80%2C%20%EC%82%AD%EC%A0%9C%2C%20%EA%B0%B1%EC%8B%A0/Pasted%20image%2020230810133244.png)

다시 테이블을 초기화하고 아래와 같은 상태로 만들어주었습니다.

![[Pasted image 20230810133213.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/4%EC%9E%A5%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%94%EA%B0%80%2C%20%EC%82%AD%EC%A0%9C%2C%20%EA%B0%B1%EC%8B%A0/Pasted%20image%2020230810133213.png)

다음은 2번 UPDATE 명령의 결과입니다. 
`UPDATE sample1 SET val1 = no, no = no + 1;`

![[Pasted image 20230810134017.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/4%EC%9E%A5%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EC%B6%94%EA%B0%80%2C%20%EC%82%AD%EC%A0%9C%2C%20%EA%B0%B1%EC%8B%A0/Pasted%20image%2020230810134017.png)

MySQL에서는 SET 구에 기술된 순서로 갱신 처리가 일어납니다. 다시 말해
`UPDATE sample1 SET no = no + 1, val1 = no;`  명령문이 있을 때 `no = no + 1` 을 처리한 뒤 `val1 = no` 를 처리합니다. 따라서 MySQL의 경우, 갱신식 안에서 열을 참조할 때는 처리 순서를 고려해야 합니다. 반면 Oracle에서는 SET구에 기술한 식의 순서가 처리에 영향을 주지 않습니다.

