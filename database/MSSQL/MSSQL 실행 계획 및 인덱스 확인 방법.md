
*[참고 : mssql SHOWPLAN_TEXT](https://learn.microsoft.com/ko-kr/sql/t-sql/statements/set-showplan-text-transact-sql?view=sql-server-ver16)*


MSSQL 에는 MYSQL 같이 `explain`  명령어가 없다. 

그래서 쿼리 실행 계획을 조회하기 위해서는 아래 설정을 해줘야 한다.

실제 실행계획이라고 한다.

```
set showplan_text ON;
```


`scan` 스캔은 처음부터 끝까지 읽는 것을 의미한다.

`seek` 는 데이터베이스에서 검색을 의미한다.  `Index Seek` 는 인덱스를 찾았다는 것이다. 뒤에어 어떤 인덱스를 찾았는지 그리고 어떻게 검색했는지가 나온다.


### 인덱스를 안탄 예시

```
Clustered Index Scan(...)
```


### 인덱스를 탄 예시

```
Index Seek(...) ORDERED FORWARD)

Clustered Index Seek(...) LOOKUP ORDERED FORWARD)
```
