
조회 방법 커맨드에서

윈도우 OS : `set`
MAC, 리눅스 OS : `printenv`


### 애플리케이션에서 OS 환경 변수 조회 하는 방법
---


```
public static void main(String[] args) {  
    Map<String, String> envMap = System.getenv();  
    for (String key : envMap.keySet()) {  
        log.info("env {}={}", key, envMap.get(key));  
    }  
}
```


*결과 일부*

![[Pasted image 20230920232620.png]](Pasted%20image%2020230920232620.png)


### 실제 활용 방법
---

로컬 환경을 제외한 개발, 운영 환경에서는  `Linux` OS 를 사용하는 경우가 많다. 따라서 **bash.bashrc**
파일 같은 곳에 환경 변수를 설정한 뒤 애플리케이션에서 불러와서 사용하자.



### OS 환경 변수 단점
---
하나의 프로그램 입장에서 OS 환경 변수는 전역 변수이다. OS 내부에 여러 프로그램이 존재하고 관리해야 한다면 전역 변수 관리에 휴먼 리소스를 필요 이상으로 사용해야할지도 모른다. 

이러한 단점을 해결하기 위해 특정 자바 프로그램안에서 사용하는 외부 설정을 알아보도록 하자.
