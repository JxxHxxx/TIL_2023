
`RECYCLEBIN` 객체에는 드랍 된 `TABLE` , `INDEX` 내역 등이 존재한다. 구글에서는 쓰레기통이라고도 표현하며 이를 통해 drop 된 테이블을 복구할 수도 있다. 우선 `RECYCLEBIN` 을 조회해보자.


```
SELECT * FROM RECYCLEBIN;
```


*결과*

![[Pasted image 20230829125228.png]]


```
FLASHBACK TABLE 테이블명 TO BEFORE DROP; // RECYCLEBIN 조회 시 ORIGINAL_NAME 과 매칭되게 입력하면 된다. 
```


복구된 테이블은 외래키 제약조건이나 `INDEX` 가 미지정될 수 있으므로 복구 후 잘 살펴보자. 

##### 테이블 제약 조건 조회

```
SELECT * FROM ALL_CONSTRAINTS
WHERE TABLE_NAME = '테이블 명';
```


##### 인덱스 조회

```
SELECT * FROM ALL_INDEXES 
WHERE TABLE_NAME = '테이블 명';
```


#### SYS 에서 recyclebin 관리

아래 명령어는 SYS 권한을 가진 계정에서 입력할 수 있는 것으로 보인다. `/as sysdba` 로 접속하자.


```
SHOW parmeter RECYCLEBIN;
```


![[Pasted image 20230829123303.png]]


`RECYCLEBIN` 활성화 옵션을 변경할 수도 있다.

```
ALTER session SET RECYCLEBIN=off;
```


실험 결과 세션 변경, 커밋, DB 재연결을 해도 `recyclebin` 이 활성화 되어 있다.

