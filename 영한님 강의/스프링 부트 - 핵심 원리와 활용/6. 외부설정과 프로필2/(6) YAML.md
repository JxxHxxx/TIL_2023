
`properties` 말고 `yaml` ,`yml` 확장자로 설정 값들을 지정할 수 있다. `yml` 의 장점은 사람이 읽기 좋게 계층 구조를 이룬다는 점이다. 아래 `properties` ,`yml` 예시를 통해 차이를 파악해보자. 
 
*application.properties*

```
my.datasource.url=local.db.com  
my.datasource.username=local_user  
my.datasource.password=local_pw  
my.datasource.etc.max-connection=10  
my.datasource.etc.timeout=10s  
my.datasource.etc.options=CACHE,ADMIN
```


*application.yml*

```
my:  
  datasource:  
    url:   
    username:  
    password:  
    etc:  
      max-connection: 10  
      timeout: 10s  
      options: CACHE,ADMIN
```



참고로 `application.properties` , `application.yml` 을 동시에 사용하면 
- `application.properties` 가 우선권을 가진다.
- 하지만 동시에 사용할 명분이나 이점이 적으니 한 확장자로 통일하는 편이 좋다.