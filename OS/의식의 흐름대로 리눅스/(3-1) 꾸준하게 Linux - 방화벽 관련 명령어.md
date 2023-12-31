
서로 다른 네트워크를 지나는 데이터를 허용하거나 거부하거나 검열, 수정하는 하드웨어나 소프트웨어 장치라고 한다. 출처[ 나무위키](https://ko.wikipedia.org/wiki/%EB%B0%A9%ED%99%94%EB%B2%BD_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9))

![[Pasted image 20230824215310.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/OS/%EC%9D%98%EC%8B%9D%EC%9D%98%20%ED%9D%90%EB%A6%84%EB%8C%80%EB%A1%9C%20%EB%A6%AC%EB%88%85%EC%8A%A4/images/Pasted%20image%2020230824215310.png)

그림을 살펴보면 동일한 LAN에서는 방화벽이 작동하지 않는다.  예를 들어 
`15.50.100.2/16` 과 `15.50.100.3/16` 은 같은 네트워크 대역이기 때문에 동일한 LAN에 속한다. 즉, 방화벽에서 관리하는 영역이 아니다.

그래서 두 서버는 서로 통신을 하는데 방화벽 상의 제한이 없다. 실제로 버추얼 머신같은 곳에 인스턴스를 하나파서 테스트해보면 방화벽 설정을 하지 않더라도 `ssh`  프로토콜을 이용할 수 있다. (버추얼 머신이 어떻게 IP를 할당하는지는 모르나 확인해보면 두 IP는 모두 동일한 네트워크 대역에 속했다.)

하지만 다른 네트워크 간에 통신을 하려면 방화벽 `port` , `ip` 를 설정해주어야 한다.


이와 관련된 명령어를 몇 개 정리하겠습니다. 

우선 방화벽 관련 명령을 사용하기 위해서는 관리자 권한이 필요합니다.


현재 방화벽 리스트 

```
firewall-cmd --list-all
```

특정 포트 접근 허용/제거하기

```
firewall-cmd --permanent --{add/remove}-port={port}/tcp
```

정상적으로 수행될 경우 `success` 를 응답한다.

특정 IP 접근 허용/제거하기
```
firewall-cmd --permanent --add-source={ip address}/{subnet-mask}
```

변경된 방화벽 구성 적용

```
firewall-cmd --reload
```


이외에도 방화벽 관련 명령어를 통해 특정 포트와 IP를 결합하여 세밀하게 접근을 허용하거나 제한할 수 있다.
