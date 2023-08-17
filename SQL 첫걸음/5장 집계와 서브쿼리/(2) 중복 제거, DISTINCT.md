
집합을 다룰 때, 경우에 따라서는 집합 안에 중복된 값이 있는지 여부가 문제될 때도 있습니다. 데이터가 서로 중복되지 않는 경우에는 '유일한 값을 가진다'라든가 '값이 중복되지 않는다.' 라는 표현을 자주 사용합니다.

DISTINCT 키워드를 통해 중복된 값을 제외하고 SELECT 할 수 있습니다. 참고로 집계 함수는 아닙니다.

아래와 같은 테이블이 존재한다고 했을 때

![[Pasted image 20230810152409.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230812164909.png)

`select DISTINCT name FROM member2;`  이런 식으로 사용할 수 있습니다.

![[Pasted image 20230810152436.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230810152436.png)

앞서 집계 함수는 집합 안에 있는 NULL 값을 제외한다고 했습니다. DISTINCT는 집계 함수가 아니기 떄문에 NULL 을 제외하지 않습니다.

