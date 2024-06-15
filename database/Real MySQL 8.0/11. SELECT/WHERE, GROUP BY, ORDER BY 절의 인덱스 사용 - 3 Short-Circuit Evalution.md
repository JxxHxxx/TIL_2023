
```java
// java
if (inTransaction && hasModified()) {
	commit();
}
```

위와 같은 코드가 있을 때  Java 에서 && 연산자 조건을 판단할 때 `inTransaction` 조건이 거짓이라면 뒤에 나오는 `hasModified()` 메서드를 수행하지 않고 넘어간다.

이것이 선행표현식의 결과에 따라 후행 표현식을 평가 여부를 결정하는 최적화 방식이고 다른 말로 `Short-Circuit Evaluation` 이라고 한다.

MySQL `WHERE` 비교에서도 이러한 최적화 방식이 적용되는 부분이 있다.

우선 `WHERE` 조건에 인덱스를 사용할 수 있는 조건이 있다면 `Short-Circuit Evaluation`  와 상관없이 옵티마이저는 해당 조건을 최우선으로 사용한다. 

정리하면 인덱스 조건을 가장 먼저 사용하고 그 다음부터는 조건을 순서대로 평가한다.

```sql
-- first_name 은 index 컬럼
SELECT * FROM employees 
WHERE last_name = 'Aamodt' 
AND first_name = 'Matt' 
AND MONTH(birth_date) = 1; 
```

위 쿼리는 인덱스인 `first_name` 을 가장 먼저 사용하고 `last_name` ,  `MONTH(birth_date)` 조건을 순차적으로 평가한다.

이 순서는 쿼리의 성능에 영향을 줄 수 있다.

예를 들어 `last_name` 조건이 레코드 하나만 만족했다면 뒤이어 나오는 `MONTH(birth_date)` 평가는 레코드 하나만 비교하면 된다.

반면  아래와 같이 쿼리가 변경됐고 `MONTH(birth_date)` 조건을 만족하는 레코드가 10000개 라고 해보자.

```sql
-- first_name 은 index 컬럼
SELECT * FROM employees 
WHERE MONTH(birth_date) = 1 
AND first_name = 'Matt' 
AND last_name = 'Aamodt'; 
```

이렇게 되면 1만개의 레코드에 대해  `last_name = 'Aamodt'` 조건이 만족하는지 평가해야 한다.