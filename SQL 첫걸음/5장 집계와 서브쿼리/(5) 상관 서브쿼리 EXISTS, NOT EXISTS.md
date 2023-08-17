
## EXISTS, NOT EXISTS

EXISTS 술어를 사용하면 서브쿼리가 반환하는 결과값이 있는지를 조사할 수 있습니다. EXISTS를 사용하는 경우에는 서브쿼리가 반드시 스칼라 값을 반환할 필요는 없습니다. EXIST는 단지 반환된 행이 있는지를 확인해보고 값이 있으면 참, 없으면 거짓을 반환하므로 어떤 패턴이라도 상관없습니다.


예시 테이블 

`select * from sample551;`

![[Pasted image 20230812173530.png]]

`select * from sample552;`

![[Pasted image 20230812173539.png]]

지금부터 sample552에 no 열의 값과 같은 행이 있다면 sample551 val 열을 '있음' 이라는 값으로 그것이 아니라면 '없음' 이라는 값으로 갱신하겠습니다.

이것을 WHERE 구를 통해 해당 요구 사항을 해결하려면 아래와 같이 쿼리를 짜면 됩니다.

```
UPDATE sample551 SET val = '있음' WHERE ...
UPDATE sample551 SET val = '없음' WHERE ...
```


문제는 행에 매우 많다면 해당 쿼리를 짜기 힘들 수 있습니다.

`EXISTS`, `NOT EXISTS` 를 활용하면 매우 효율적으로 짤 수 있습니다.

```
UPDATE sample551 SET val = '있음' WHERE  
    EXISTS(SELECT * FROM sample552 WHERE no = sample551.no);  
  
UPDATE sample551 SET val = '없음' WHERE  
    NOT EXISTS(SELECT * FROM sample552 WHERE no = sample551.no);
```


![[Pasted image 20230812174656.png]]


```
UPDATE sample551 SET val = '있음' WHERE  
    EXISTS(SELECT * FROM sample552 WHERE no = sample551.no);  
```

해당 쿼리에서는 UPDATE 명령(부모)에서 WHERE 구에 괄호에 묶은 부분(EXISTS ~)이 서브쿼리(자식)이 됩니다.  이때 자식인 서브쿼리는 부모 명령에서의 테이블과 no 열 값과 일치하는 행을 검색합니다. 

이처럼 부모 명령과 자식인 서브쿼리가 **특정 관계**를 맺는 것을 '상관 서브쿼리'라 부릅니다.