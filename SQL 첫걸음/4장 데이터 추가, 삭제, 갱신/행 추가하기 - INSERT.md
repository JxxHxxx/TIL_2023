이 장에서는 데이터를 추가하는 방법을 알아보도록 하겠습니다. 데이터베이스의 테이블에 행을 추가하기 위해서는 INSERT 명령을 사용합니다.

문법
`INSERT INTO 테이블명 VALUES (값1, 값2, ...)`

## 목차

1. [INSERT로 행 추가하기]
2. [NOT NULL 제약]
3. [DEFAULT]

## INSERT로 행 추가하기

RDBMS에서는 INSERT 명령을 사용해 테이블의 데이터를 추가할 수 있습니다.

`DESC study_groups`

![[Pasted image 20230809222938.png]]


```
INSERT INTO study_groups (name, type, created_at)  
    VALUES ('자바의 신', 'JAVA', '2023-08-09');
```


가장 하단을 보면 INSERT 명령을 통해 데이터가 잘 저장됐음을 확인할 수 있습니다.

![[Pasted image 20230809223228.png]]


## NOT NULL 제약


앞서 보았던 study_groups 테이블의 모든 컬럼들은 Null 값을 혀용하지 않습니다. 한 번 NULL 값이 허용되는 테이블에 값을 지정하여 데이터를 넣어보도록 하겠습니다.

![[Pasted image 20230809223532.png]]


```
INSERT INTO items_unit (price, quantity)  
    VALUES (2000, 30);
```

데이터를 저장한 후 조회하면 아래와 같이 unit 컬럼이 `null` 값으로 존재하는 것을 볼 수 있습니다.

![[Pasted image 20230809223630.png]]


따라서 특정 컬럼이 NULL 값을 가지길 원하지 않는다면 NOT NULL 제약 조건을 사용해야 합니다.

제약은 아래와 같이 테이블을 생성할 때나 컬럼을 수정할 때 사용할 수 있습니다.

```
CREATE TABLE study_groups (  
      group_id BIGINT PRIMARY KEY AUTO_INCREMENT,  
      name VARCHAR(255) NOT NULL,  
      type VARCHAR(255) NOT NULL,  
      created_at DATE NOT NULL  <- NULL 불허 제약
);
```


## DEFAULT

Default는 명시적으로 값을 지정하지 않았을 경우 사용하는 초깃값을 말합니다.

```
CREATE TABLE dummy_1 (  
    id bigint primary key,  
    val int default 0  
);
```

위와 같은 테이블 스키마가 존재할 때

명시적으로 defalut 값을 설정할 수도 있습니다.

`insert into dummy_1 (id, val) values (1, default);` 

물론 생략할 수도 있습니다.

`insert into dummy_1 (id) values (2);`