`JobRepository` 란, 스프링 배치 잡과 관련된 데이터 저장소이다. 

`JobRepository 는 두 가지 데이터 저장소를 제공한다.

- 인메모리
- RDBMS


##### 관계형 데이터베이스
---

![[Pasted image 20231214230647.png]]

출처 : [spring.io batch project](https://docs.spring.io/spring-batch/docs/4.3.10/reference/html/index-single.html#metaDataSchemaOverview)


##### BATCH_JOB_INSTANCE 테이블
---
이 스키마의 실제 시작점은 `BATCH_JOB_INSTANCE` 이다. 잡을 식별하는 고유 정보가 포함된 잡 파라미터로 잡을 처음 실행하면 단일 `JobInstance` 레코드가 테이블에 등록된다.

- JOB_EXECUTION_ID : 테이블의 기본 키
- VERSION : 낙관적 락에 사용되는 레코드 버전
- JOB_NAME : 실행된 잡의 이름
- JOB_KEY : 잡 이름과 잡 파라미터의 해시 값으로, `JobInstance` 를 고유하게 식별하는데 사용되는 값

##### BATCH_JOB_EXECUTION 테이블
---
이 테이블은 배치 **잡의 실제 실행 기록**을 나타낸다. 잡이 실행될 때마다 새 레코드가 해당 테이블에 생성되고, 잡이 진행되는 동안 주기적으로 업데이트된다.

테이블이 가지는 컬럼은 아래와 같다

- JOB_EXECUTION_ID : 테이블의 기본 키
- VERSION : 낙관적인 락에 사용되는 레코드 버전
- JOB_INSTANCE_ID : BATCH_JOB_INSTANCE 테이블을 참조하는 외래 키
- CREATE_TIME : 레코드가 생성된 기간
- START_TIME : 잡 실행이 시작된 시간
- END_TIME : 잡 실행이 완료된 시간
- EXIT_CODE : 잡 실행의 종료 코드
- EXIT_MESSAGE : EXIT_CODE와 관련된 메시지나 스택 트레이스
- LAST_UPDATE : 레코드가 마지막으로 갱신된 시간

##### BATCH_JOB_EXECUTION_CONTEXT 테이블
---
`BATCH_JOB_EXECUTION` 테이블과 연관이 있는 테이블 중 하나이다. `ExecutionContext` 를 통해 배치 잡의 상태를 저장할 수 있다. 이 테이블은 `ExecutionContext` 를 저장하는 곳이다.

테이블이 가지는 컬럼은 아래와 같다

- JOB_EXECUTION_ID : 테이블의 기본 키
- SHORT_CONTEXT : 트림 처리된 SERIALIZED_CONTEXT
- SERIALIZED_CONTEXT : 직렬화된 ExecutionContext

기본 설정을 가져가게 된다면 `SERIALIZED_CONTEXT` 컬럼은 null 값을 가진다. 이를 커스터마이징하는 방법은 5장의 뒷부분에서 설명한다.


##### BATCH_JOB_EXECUTION_PARMAS 테이블
---
`BATCH_JOB_EXECUTION` 테이블과 연관이 있는 두번째 테이블이다. 이 테이블에는 잡이 매번 실행될 때마다 사용된 잡 파라미터를 저장한다. 앞서 잡의 식별 정보가 담긴 잡 파라미터를 사용해 새`JobInstnace` 가 필요한지 결정하는 방법을 살펴봤다. 

그러나 실제로는 잡에 전달된 모든 파라미터가 테이블에 저장된다. 그리고 재시작시에는 잡의 식별 정보 파라미터만 자동으로 전달된다.

테이블이 가지는 컬럼은 아래와 같다

- JOB_EXECUTION_ID : 테이블의 기본 키
- TYPE_CODE : 파라미터 값의 타입을 나타내는 문자열
- KEY_NAME : 파라미터 이름
- STRING_VAL : 타입이 String인 경우 파라미터의 값
- DATE_VAL : 타입이 SDate인 경우 파라미터의 값
- DOUBLE_VAL : 타입이 Double인 경우 파라미터의 값
- LONG_VAL : 타입이 Long인 경우 파라미터의 값
- IDENTIFYING : 파라미터가 식별되는지 여부를 나타내는 플래그


##### BATCH_STEP_EXECUTION
---
이 테이블에는 스탭의 시작, 완료, 상태에 대한 메타데이터를 저장한다.  또한 스텝 분석이 가능하도록 다양한 횟수 값을 추가로 저장한다. 또한 읽기(`read`), 처리(`process`), 쓰기(`write`) 횟수, 건너뛰기(`skip`) 횟수 등과 같은 모든 데이터가 저장된다.

테이블이 가지는 컬럼은 아래와 같다

- STEP_EXECUTION_ID : 테이블의 기본 키
- VERSION : 낙관적인 락에 사용되는 레코드의 버전
- STEP_NAME : 스탭 이름
- JOB_EXECUTION_ID : BATCH_JOB_EXECUTION 테이블을 참조하는 외래 키
- START_TIME : 스탭 실행이 시작된 시간
- END_TIME : 스탭 실행이 완료된 시간
- STATUS : 스탭의 배치 상태
- COMMIT_COUNT : 스탭 실행 중에 커밋된 트랜잭션 수
- READ_COUNT : 읽은 아이템 수
- FILTER_COUNT : 아이템 프로세서가 `null`을 반환해 필터링된 아이템 수
- WRITE_COUNT : 기록된 아이템 수
- READ_SKIP_COUNT : `ItemReader` 내에서 예외가 던져졌을 때 건너뛴 아이템 수
- PROCESS_SKIP_COUNT : `ItemProcessor` 내에서 예외가 던져졌을 때 건너뛴 아이템 수
- WRTIE_SKIP_COUNT : `ItemWriter` 내에서 예외가 던져졌을 때 건너뛴 아이템 수
- ROLLBACK_COUNT : 스텝 실행에서 롤백된 트랜잭션 수
- EXIT_CODE : 스텝의 종료 코드
- EXIT_MESSAGE : 스텝 실행에서 반환된 메시지나 스택 트레이스
- LAST_UPDATED : 레코드가 마지막으로 업데이트된 시간


#####  BATCH_STE_EXECUTION_CONTEXT 테이블
---
이 테이블은 `StepExecution` 에서 사용되는 `ExectuionContext` 가 존재하는데 이에 대한 정보를 저장한다. 다시 말해 `StepExecution` 의 `ExecutionContext`는 스텝 수준에서 컴포넌트의 상태를 저장하는데 사용된다. `ItemReader` , `ItemWriter` 같은 컴포넌트를 자세히 살펴볼 때 해당 테이블의 사용법을 더욱 자세히 알아볼 것이다.

테이블이 가지는 컬럼은 아래와 같다

- STEP_EXECUTION_ID : 테이블의 기본 키
- SHORT_CONTEXT : 트림 처리된 SERIALIZED_CONTEXT
- SERIALIZED_CONTEXT : 직렬화된 ExecutionContext

