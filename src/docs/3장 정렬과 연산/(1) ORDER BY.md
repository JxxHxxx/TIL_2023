`SELECT` 명령의 `ORDER BY` 구를 사용하여 검색 결과의 행 순서를 바꿀 수 있습니다. 여기서는 정렬 방법을 배워보겠습니다.

## 목차

1. [정렬 - ORDER BY](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/(1)%20ORDER%20BY.md#%EC%A0%95%EB%A0%AC---order-by)
2. [복수의 열을 지정해 정렬하기](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/(1)%20ORDER%20BY.md#%EB%B3%B5%EC%88%98%EC%9D%98-%EC%97%B4%EC%9D%84-%EC%A7%80%EC%A0%95%ED%95%B4-%EC%A0%95%EB%A0%AC%ED%95%98%EA%B8%B0)

## 정렬 - ORDER BY

문법
`SELECT {column_name_1} FROM {table_name} WHERE {condition} ORDER BY {column_name_2}` 

ORDER BY 구를 지정하지 않을 경우에는 데이터베이스 내부에 저장된 순서로 반환됩니다. 언제나 정해진 순서로 결괏값을 얻기 위해서는 ORDER BY구를 지정해야 합니다.

내림차순 정렬 (DESC), 오름차순 정렬(ASC or 생략)

`select * from study_groups order by created_at desc;`

![[Pasted image 20230808161712.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808161712.png)

보시는 것과 같이 내림차순으로 레코드가 정렬된 것을 볼 수 있습니다. 내림(오름) 차순에 관한 자세한 내용은 아래 대소 관계에서 추가적인 설명을 하겠습니다.

### 대소 관계

정렬에 있어 대소 관계는 중요한데요. DESC는 **하강**이라는 의미입니다. 큰(대) 값에서 작은(소) 값으로 하강한다는 의미인데요. 자료형에 따라 어떠한 대소 관계를 가지는지 알아봅시다.

- 수치형 : 수치형은 일반적으로 알고 있는 것과 동일합니다. `1 < 2 < 10 < 100` 처럼 값이 클수록 크다고 판별합니다.

- 날짜시간형 : 저는 날짜 대소 관계를 따질 때 조금 헷갈렸는데요. 날짜시간형의 대소 관계는 다음과 같습니다. `1999년 < 2013년 < 2014년` 즉, 최신일수록 값이 크다고 판별합니다. 

- 문자열형 : 문자열 자료형은 사전식 순서로 대소를 판별합니다. `apple` 보다는 `banana` 가 더 크다고 판별합니다.  `가방` 보다 `다람쥐` 가 더 크다고 판별합니다. 쉽게 사전을 펼쳤을 때 뒤에 있는 문자열이 더 크다고 판별합니다. 예시를 하나 보겠습니다.
   `select * from study_groups order by name desc;`

	![[Pasted image 20230808162454.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808162454.png)

 결과를 보면 알 수 있듯이 사전 순으로 정렬된 것을 볼 수 있습니다. 그리고 알파벳보다 한글이 더 큰 값을 가지는 것도 볼 수 있네요. (이 부분은 뭔가 데이터베이스 설정에 따라 바꿀 수 있을 것 같은데요?)


### 사전 식 순서에서 주의할 부분
문자열형에는 숫자 데이터를 넣을 수 있습니다. 이 때에는 사전 식 정렬 방식을 따르므로 주의해야 합니다. 예시를 살펴봅시다.

`select *from string_numbers order by val;`

![[Pasted image 20230808163225.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808163225.png)

- 더 알아보기
```
우리가 아는 일반적인 수치형 정렬 방식이라면 1 , 2, 10, 11 순으로 정렬되어야 하지만 
사전식 정렬 방식을 따르기 때문에 다른 결과로 정렬됩니다. 
참고로 이러한 결과가 나온 이유는 수치형 데이터를 문자열로 나타내는 경우 
숫자 전체를 보고 판단하는 것이 아니라 한 자리씩 비교하기 때문입니다.
```

### ORDER BY는 테이블에 영향을 주지 않는다.

마지막으로 ORDER BY는 테이블에 영향을 주지 않습니다. 즉 클라이언트에게 행 순서를 바꾸어 결과를 반환하는 것 뿐, 저장장치에 저장된 데이터의 행 순서를 변경하는 것은 아닙니다.


## 복수의 열을 지정해 정렬하기

ORDER BY로 행을 정렬하는 경우 같은 값을 가진 행의 순서는 **일정하지 않**습니다. 따라서 언제나 같은 순서로 결과를 얻고 싶다면 반드시 ORDER BY 구로 순서를 지정해야 합니다.

아래와 같은 테이블 존재합니다.

![[Pasted image 20230808165137.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808165137.png)

학년, 학급에 알맞게 정렬한다고 해봅시다. (1-1, 1-2, 1-3, 2-1, 2-2 이런 순으로...) 대강 생각해봐도 grade 만으로는 원하는 결과를 얻을 수 없을 것 같습니다. 이럴 때는 1개 이상의 열을 정렬 기준으로 지정합니다.

`select * from school_class order by grade, class;`

![[Pasted image 20230808165242.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808165242.png)

`order by` 뒤에 지정한 각각의 열에는 `asc` 정렬 기준이 생략되었습니다. 

select * from school_class order by grade desc, class asc; 이런 식으로 정렬 방식을 명시할 수도 있습니다.

마지막으로 ORDER BY 구를 사용할 때 주의할 점은 열을 지정하는 순서입니다.

앞서 `select * from school_class order by grade, class;` SQL을 
`select * from school_class order by class, grade;` 으로 변경해보겠습니다.

![[Pasted image 20230808165658.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808165658.png)

보시는 것과 같이 `class` 열을 먼저 정렬한 뒤 `grade`열을 정렬합니다. 즉, ORDER BY 구 뒤에 지정한 열의 순서대로 정렬을 진행합니다.

## NULL 값 정렬하기

`NULL` 은 특성상 대소비교를 할 수 없습니다. 그래서 정렬 시에는 별도의 방법으로 취급합니다. '특정 값보다 큰 값' , '특정 값보다 작은 값' 의 두 가지로 나뉘며 이 중 하나의 방법으로 대소를 비교합니다. 쉽게 말해서 가장 먼저 표시되거나 가장 나중에 표시됩니다.

표준 SQL에도 규정되어 있지 않기 때문에 데이터베이스마다 그 기준이 다릅니다. MySQL의 경우 NULL 값을 가장 작은 값으로 취급합니다. 즉 DESC(내림차순)에서는 가장 나중에 표시되고 ASC(오름차순)에서는 가장 먼저 표시됩니다.

예시
`select *from school_class order by grade desc;`

![[Pasted image 20230808170205.png]](https://github.com/JxxHxxx/sql-master/blob/master/src/docs/3%EC%9E%A5%20%EC%A0%95%EB%A0%AC%EA%B3%BC%20%EC%97%B0%EC%82%B0/Pasted%20image%2020230808170205.png)
