
리눅스 배포판은 FHS라는 표준에 따른 디렉토리 구성을 권장합니다.


![[Pasted image 20230820152258.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/OS/%EC%9D%98%EC%8B%9D%EC%9D%98%20%ED%9D%90%EB%A6%84%EB%8C%80%EB%A1%9C%20%EB%A6%AC%EB%88%85%EC%8A%A4/images/Pasted%20image%2020230820152258.png)

각각의 디렉터리는 다음을 의미한다.

```
bin : 일반사용자용 명령어
boot : 부트로더와 부팅파일
dev : 장치파일
etc : 시스템, 프로그램의 환경설정 파일
lib : 공유 라이브러리와 커널 모듈
media : 이동식 디스크의 마운트
mnt : 파일시스템의 임시 마운트
opt : 응용프로그램
sbin : 시스템 관리 명령어
srv : 시스템이 제공하는 서비스 파일
tmp : 임시파일
usr : 응용프로그램
var: 시스템 운용중 자주 변경되는 파일
```
