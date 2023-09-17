

## IN

스칼라 값끼리 비교할 때는 = 연산자를 사용합니다. 다만 **집합을 비교할 때는 사용할 수 없습니다.** IN을 사용하면 집합 안의 값이 존재하는지를 조사할 수 있습니다. IN을 사용하면 집합 안의 값이 존재하는지를 조사할 수 있습니다.

서브쿼리를 사용할 때 IN을 통해 비교하는 경우도 많아 상관 서브쿼리에서 IN 명령어를 정리하겠습니다.

앞서 테이블에서 val 열의 값이 '있음' 인 행을 조회하려면 어떻게 할 수 있을까요?

![[Pasted image 20230812174656.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230812174656.png)

OR를 이용한다면 아래와 같이 조회할 수 있습니다.
`SELECT * FROM sample551 WHERE no = 3 OR no = 5;`

하지만 선택해야될 행이 많아진다면 다른 방법을 고려해봐야 할 수도 있습니다. (쉽게 val = '있음' 조건을 거는게 제일 간단하긴 합니다.)

하지만 때에 따라 아래와 같인 IN 명령어를 사용하는게 효율적일 수 있습니다.

`SELECT * FROM sample551 WHERE no IN (3, 5);`

그리고 이 집합 부분을 서브 쿼리로도 지정할 수 있습니다.

`SELECT * FROM sample551 WHERE no IN (SELECT no FROM sample552) `


### IN과 NULL

집계함수에서는 집합 안의 NULL 값을 무시하고 처리했습니다. IN에서는 집합 안에 NULL 값이 있어도 무시하지는 않습니다. 다만 NULL = NULL을 제대로 계산할 수 없으므로 IN을 사용해도 NULL 값은 비교할 수 없습니다. 즉 NULL 을 비교할 떄는 IS NULL을 사용해야 합니다.

또한 NOT IN의 경우, 집합 안에 NULL 값이 있으면 설령 왼쪽 값이 집합 안에 포함되어 있지 않아도 참을 반환하지는 않습니다. 그 결과는 불명(UNKNOWN)이 됩니다.

즉 `SELECT * FROM sample551 WHERE no IN (3, 5) OR no is null;` 이와 같이 `is null` 을 사용해야 합니다.

다르게 말하면 아래와 같은 테이블이 있을 때 NULL = NULL 을 계산할 수 없기 때문에

*sample551*
![[Pasted image 20230812191359.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230812191359.png)

*sample552*
![[Pasted image 20230812191324.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230812191324.png)


`SELECT * FROM sample551 WHERE no IN (SELECT no FROM sample552);` 의 결과에서 `null` 을 가진 열이 반환되지 않습니다.

![[Pasted image 20230812191508.png]](https://github.com/JxxHxxx/TIL/blob/master/SQL%20%EC%B2%AB%EA%B1%B8%EC%9D%8C/5%EC%9E%A5%20%EC%A7%91%EA%B3%84%EC%99%80%20%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC/Pasted%20image%2020230812191508.png)

