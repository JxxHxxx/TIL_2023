
##### 날짜 비교 DATE, DATETIME과 문자열 비교
---

```sql
-- hire_date 는 DATETIME 타입 및 인덱스 설정
SELECT * FROM employees WHERE hire_date > '2011-07-23'
```

위와 같이 `DATETIME` 을 문자열과 비교하더라도 MySQL이 내부적으로 문자열을 `DATETIME` 타입으로 변환을 수행한다.  `DATE` 타입도 마찬가지이다.

**따라서 모두 인덱스를 효율적으로 이용한다.**

반면 아래와 같이 인덱스가 설정된 컬럼의 원본을 변경하면 인덱스를 사용할 수 없다.
```sql
SELECT * FROM employees WHERE DATE_FORMAT(hire_date, '%Y-%m-%d') > '2011-07-23';
```


##### DATE와 DATETIME 비교
---
- `DATE` 와 `DATETIME` 타입 비교에서 타입 변환은 인덱스에 영향을 미치지 않는다.
- 두 타입 간 타입을 변환하지 않으면 `DATE` 타입이 `DATETIME` 타입으로 변환된다. 즉, `2011-06-30` 과 `2011-06-30 00:00:01` 을 비교하면 앞의 `2011-06-30` 이 `2011-06-30 00:00:00` 으로 변환된다.


##### DATETIME과 TIMESTAMP 비교
---
`TIMESTAMP` 는 사실상 단순 숫자 값이다. 그래서 적절한 비교를 할 수 없다. 이때는 두 타입을 명시적으로 동일한 타입으로 변환해야 한다.

```sql
-- hire_date 는 DATETIME 타입 및 인덱스 설정
SELECT * FROM employees WHERE hire_date > FROM_UNIXTIME(UNIX_TIMESTAMP())
```