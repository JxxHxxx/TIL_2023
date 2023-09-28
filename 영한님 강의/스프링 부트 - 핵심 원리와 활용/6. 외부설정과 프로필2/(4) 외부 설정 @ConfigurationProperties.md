

`Environment` 사용 시 주의 사항은 의도한대로 타입을 지정해야 한다는 것이다. 이로 인해 컴파일 에러가 발생할 수 있다. `@ConfigurationProperties` 를 활용하면 `type-safe` 하게 설정 속성을 지정할 수 있다.

```
String port = env.getProperty("my.datasource.port"); // 실제로는 int
```


더 정확하게는 스프링 외부 설정의 묶음 정보를 **객체**로 변환할 수 있다. 이에 대한 부가 효과가 `type-safe` 이고 객체이기 때문에 활용할 수 있는 부분이 많아진다.


### @ConfigurationProperties
---

다음은 `@ConfigurationProperties` 를 활용하는 방법이다. 우선 외부 설정 파일의 값을 불러와야 하므로 `application.properties` 파일을 살펴보자.

*application.properties*

```
my.datasource.url=local.db.com  
my.datasource.username=local_user  
my.datasource.password=local_pw  
my.datasource.etc.max-connection=1  
my.datasource.etc.timeout=3500ms  
my.datasource.etc.options=CACHE,ADMIN
```


다음은 설정 파일 값을 가져와 객체로 사용하는 클래스이다.

```
@Getter @Setter // (1)
@ConfigurationProperties("my.datasource")  // (2)
public class MyDataSourcePropertiesV1 {  
  
    private String url;  
    private String username;  
    private String password;  
    private Etc etc;  // (3)
  
    @Getter @Setter  
    public static class Etc {  
        private int maxConnection;  
        private Duration timeout;  
        private List<String> options = new ArrayList<>();  
    }  
}
```


`(1)` : 자바빈 프로퍼티 접근법을 기본으로 사용한다고 한다. 
-  *참고로 생성자로 주입받을 수도 있다. *
 

`(2)` :  `@ConfigurationProperties` 외부 설정 값을 주입받아 객체로 사용할 때 필요한 애노테이션이다. 파라미터로는 설정 key의  `root` 를 지정하는 것이다. 아래 예시를 살펴보자.

파리미터로 `my.datasource` 를 사용하면 필드 값으로 `url` 을 사용할 수 있다. 

```
my.datasource.url=local.db.com  
my.datasource.etc.options=CACHE,ADMIN
```


`(3)` : 위에서 `url` 을 필드로 두어 사용할 수 있었다.  `etc.options` 는 객체로 만들어 사용하면 된다. 


### @EnableConfigurationProperties
---

앞서 `@ConfigurationProperties` 를 선언하여 생성한 설정 값들이 지정된 객체를 사용하기 위해서는 `@EnableConfigurationProperties` 를 통해 주입해야 한다.


```
@EnableConfigurationProperties(MyDataSourcePropertiesV1.class)  
public class MyDataSourceConfigV1 {  
  
    private final MyDataSourcePropertiesV1 properties;  
  
    public MyDataSourceConfigV1(MyDataSourcePropertiesV1 properties) {  
        this.properties = properties;  
    }  
  
    @Bean  
    public MyDataSource dataSource() {  
        MyDataSourcePropertiesV1.Etc etc = properties.getEtc();  
  
        return new MyDataSource(...) 
    }  
}
```



### @ConfigurationPropertiesScan
---

앞서 `@EnableConfigurationProperties` 를 사용해서 설정 속성 값을 불러와 빈을 등록하는 방식 대신 사용할 수 있는 방법이다. `@ComponentScan` 와 유사하게 `@ConfigurationProperties` 이 선언된 클래스를 스캔하여 빈으로 주입하는 기능을 한다.



### 생성자를 통한 값 주입
---

앞서 설정 값을 빈에 주입할 때 게터/세터를 이용했다. 생성자를 통해서도 주입할 수 있다고 했다. 생성자를 사용했을 때 장점은 값이 변경될 가능성을 차단할 수 있다는 점이다. 누군가가 변경하면 안되는 값을 세터를 통해 변경한다고 생각해보면 생성자로 주입하는게 어떤 이점을 가지는지 이해할 수 있을 것이다.


*사용 방법*

```
@Getter  
@ConfigurationProperties("my.datasource")  
public class MyDataSourcePropertiesV1 {  
  
    private String url;  
    private String username;  
    private String password;  
    private Etc etc;  
  
    @ConstructorBinding  // (1)
    public MyDataSourcePropertiesV1(String url, String username, String password, Etc etc) {  
        this.url = url;  
        this.username = username;  
        this.password = password;  
        this.etc = etc;  
    }
```


`(1)` :  생성자로 설정 값을 바인딩 할 때 사용하는 애노테이션 부트 3.0 이전 버전에서는 무조건 명시해야 한다. 3.0 이후 버전에서는 생성자 메서드가 하나일 때는 생략 가능 / 3.2 버전 이후 패키지가 변경되기 때문에 *2023-09-28 기준 Deprecated 선언 되어 있음*   