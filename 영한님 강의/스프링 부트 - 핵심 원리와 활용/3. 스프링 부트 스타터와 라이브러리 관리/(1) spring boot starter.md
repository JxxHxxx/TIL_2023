
##### **라이브러리 관리의 어려움**

프로젝트를 시작할 때 다양한 라이브러리가 필요하다. 예를 들어 내장 톰캣, 스프링 WEB, 로그 관련 라이브러리가 필요하다. 문제는 **라이브러리 간 호환성**이다.

이런 식으로 뒤에 버전 정보를 명시해야 한다.

```
implementation 'org.springframework:spring-webmvc:6.0.4'  
implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.5'  
implementation 'org.springframework.boot:spring-boot:3.0.2'  
implementation 'org.springframework.boot:spring-boot-autoconfigure:3.0.2' 
implementation 'ch.qos.logback:logback-classic:1.4.5'  
implementation 'org.apache.logging.log4j:log4j-to-slf4j:2.19.0'  
implementation 'org.slf4j:jul-to-slf4j:2.0.6'  
```


하지만 스프링 부트의 버전 관리 기능을 사용하면 버전 정보를 생략할 수 있다.

`build.gradle` `plugin`에 다음을 추가하자.

```
id 'io.spring.dependency-management' version '1.1.0'
```

그리고 버전 명을 제거한 채 다시 빌드를 해도 정상적으로 애플리케이션이 켜질 것이다.

*참고 - 스프링 부트가 관리하는 라이브러리 버전 목록*

https://spring.io/projects/spring-boot#learn 를 참고하여 사용중인 부트 버전을 찾자. 이후 `Dependency Versions` 에서 부트에서 관리하는 라이브러리 버전을 확인할 수 있다.

![[Pasted image 20230913230122.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/%EC%98%81%ED%95%9C%EB%8B%98%20%EA%B0%95%EC%9D%98/%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20-%20%ED%95%B5%EC%8B%AC%20%EC%9B%90%EB%A6%AC%EC%99%80%20%ED%99%9C%EC%9A%A9/3.%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20%EC%8A%A4%ED%83%80%ED%84%B0%EC%99%80%20%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%20%EA%B4%80%EB%A6%AC/Pasted%20image%2020230913230122.png)

참고로 스프링에서 모든 라이브러리를 관리하지 않는다. 관리되지 않는 라이브러리는 버전 명을 명시해야 한다.

*예제*

```
implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
```


---
### 스프링 부트 스타더


앞서 `io.spring.dependency-management` 플러그인을 통해 버전 관리를 비교적 과거보다 쉽게 할 수 있었다. 하지만 작은 프로젝트를 시작하려해도 일일히 추가해야 하는 라이브러리가 너무나도 많다.

이런 문제를 해결하기 위해 스프링 부트 스타터가 출시되었다.

이제는 여러 라이브러리를 추가할 필요 없이 아래 프로젝트만 의존하면 된다. 물론 `web` 프로젝트를 만들 때 한정이다.

```
implementation 'org.springframework.boot:spring-boot-starter-web'
```


만약 특정 라이브러리의 버전을 변경하고 싶다면 `build.gradle` 에 아래처럼 버전 명을 명시하면 된다.


```
ext['tomcat.version'] = '10.1.4'
```

참고로 `ext[]` 안에 들어가는 속성 값은 [Version Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#appendix.dependency-versions.properties) 를 참고하면 된다.
