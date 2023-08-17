이 장에서는 GROUP BY 구를 사용해 그룹화하는 방법을 정리하겠습니다.

`SELECT * FROM {table_name} GROUP BY {column1}, {column2}, ...`

아래와 같은 테이블이 존재했을 때

![[Pasted image 20230810155827.png]]


GROUP BY 를 통해 그룹을 나누고 집계 합수를 조합할 수 있습니다.
`select name, sum(quantity) from sample51 GROUP BY name;`

![[Pasted image 20230810160237.png]]


## HAVING 구로 조건 지정

아래 SQL을 실행되지 않습니다. 집계 함수는 WHERE 구의 조건식에서 사용할 수 없기 때문입니다. `WHERE sum(quantity) > 5` <- 이 부분

```
SELECT name, sum(quantity)  
FROM sample51  
WHERE sum(quantity) > 5 
GROUP BY name;
```


에러가 발생하는 이유는 GROUP BY, WHERE 구의 내부 처리 순서와 관계 있습니다.

```
내부 처리 순서

WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```

책에서는 GROUP BY 라고 말했지만 아마도 집계 함수가 WHERE 이후에 실행된다는 걸 전달하고 싶었던 것 같습니다?!

이러한 문제를 해결하려면 HAVING 구를 사용하면 됩니다.

```
SELECT name, sum(quantity)  
FROM sample51  
GROUP BY name  
HAVING sum(quantity) > 5;
```


이전과는 달리 결과를 볼 수 있습니다.

![[Pasted image 20230810162404.png]]


## 복수열의 그룹화

GROUP BY에 **지정한 열 이외**의 열은 **집계함수를 사용하지 않은 채** SELECT 구에 기술해서는 안됩니다.

```
SELECT name, quantity, sum(quantity) FROM sample51  
GROUP BY name;
```

다시 말해 위 명령에서 1. GROUP BY에도 지정되지 않고 2. 집계함수를 사용하지도 않은

quantity 열은 사용할 수 없습니다. 데이터베이스에 따라 에러가 발생할 수 있습니다. MySQL에서는 아래와 같은 에러가 발생합니다.

![[Pasted image 20230810163221.png]]

에러가 발생하는 이유는 name 컬럼은 GROUP BY에 의해 그룹핑되었지만 quantity는 그루핑되지 않았습니다. 이에 따라 어떠한 열 값을 반환해야 하는지 데이터베이스는 판단할 수 없습니다.

