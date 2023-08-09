`SELECT` 명령에서는 결괏값으로 반환되는 행을 제한할 수 있습니다. 여기서는 `LIMIT` 구로 결과 행을 제한하는 방법에 관해 알아보겠습니다.

## 목차

1. [행수 제한]


## 행수 제한

LIMIT 구는 표준 SQL은 아닙니다. MySQL, PostgreSQL에서 사용할 수 있는 문법입니다. LIMIT 구는 SELECT 명령의 마지막에 지정하는 것으로 WHERE 구, ORDER BY 구 뒤에 지정합니다.

문법
`SELECT {column_name_1} FROM {table_name} WHERE {condition} ORDER BY {column_name_2} LIMIT {row_max}`

*더 알아보기 LIMIT 동작 방식*
```
MySQL에서 SELECT 절의 실행 순서는 다음과 같습니다. 
WHERE 및 JOIN -> GROUP BY -> DISTNCT -> HAVING -> ORDER BY -> LIMIT

즉 WHERE 조건, ORDER BY 정렬을 모두 수행한 뒤에 최종적으로 LIMIT 을 수행합니다. 
조금 더 자세한 내용은 Real MySQL 2권 page 78 을 참고하길 바랍니다.
```


### 오프셋(OFFSET) 지정

게시판 같은 웹 서비스를 이용할 때 페이지 단위로 화면에 표시할 컨텐츠를 처리합니다. 대량의 데이터를 하나의 페이제 표시하는 것은 기능적으로도 속도 측면에서도 효율적이지 못할 수 있습니다. 그래서 일반적으로 페이지 나누기(pagination) 기능을 사용합니다.

이 페이지 나누기 기능은 LIMIT을 사용해 간단히 구현할 수 있습니다. 아래와 같이 SRPING 관련 서적을 조회한다고 합시다.

`select * from study_groups where type = 'SPRING';`

![[Pasted image 20230808174104.png]]

한 페이지 내부에 서적을 2개씩만 표현한다고 했을 때 아래와 같이 쿼리를 작성할 수 있습니다.

`select * from study_groups where type = 'SPRING' limit 2;`

![[Pasted image 20230808174213.png]]

이제 첫 번째 페이지에 표시할 서적들을 표현할 수 있게 되었습니다. 2번 째 페이지를 조회하기 위해서는 어떻게 해야할까요? 이 때 유용한 문법이 OFFSET입니다. OFFSET은 OO부터 라는 의미를 가집니다. OFFSET의 시작 값은 0 입니다. 

위 쿼리에 따라 OFFSET 값이 0, 1인 레코드를 불러온 것입니다. 이제 2번째 페이지를 불러오려면 어떡해야 할까요? OFFSET 이 2인 레코드부터 불러오면 될 것입니다.

`select * from study_groups where type = 'SPRING' limit 2 offset 2;`

![[Pasted image 20230808174444.png]]

