하이버네이트는 자바 진영의 `ORM` 기술이다. 하지만 배치 작업에서 하이버네이트를 사용할 때 주의해야할 두 가지 문제가 있다.

- `PersistenceContext` 
- 성능 문제

###### PersistenceContext

하이버네이트의 기본 세션은 `stateful` 이다. 그래서 아이템을 백만 건 읽고 처리한 뒤 동일하게 백만 건을 쓴다면 하이버네이트 세션이 데이터베이스에서 조회할 때 아이템을 
캐시(`PersistenceContext`)에 쌓게 된다. 이 과정에서 `OutOfMemoryException` 이 발생할 수도 있다.

###### 성능 문제

직접 JDBC를 사용할 때보다 하이버네이트를 사용할 때 더 큰 부하를 유발한다는 것이다. 대용량 데이터를 처리할 때는 `ms` 단위의 차이가 거대한 차이를 만든다.

스프링 배치에서는 이 문제를 해결하기 위해 `ItemReader` 가 커밋할 때 세션을 플러시하고 있다.


##### 하이버네이트로 커서 처리하기

---

우선 `BatchConfigurer` 를 상속받아 하이버네이트 구현체에 알맞는 `TransactionManager` 를 주입해주자.  스프링 배치는 기본적으로 `TransactionManager` 에 `DataSourceTransactionManager` 를 주입한다. 사실 스프링 부트 jpa 를 의존 받으면 자동으로 `HibernateTransactionManager` 를 주입해주긴 한다. 여기서는 직접 구성해보도록 하자.

```
@Component  
public class HibernateBatchConfigurer extends DefaultBatchConfigurer {  
  
    private DataSource dataSource;  
    private SessionFactory sessionFactory;  
    private PlatformTransactionManager transactionManager;  
  
    public HibernateBatchConfigurer(DataSource dataSource, EntityManagerFactory entityManagerFactory) {  
        super(dataSource);  
        this.dataSource = dataSource;  
        this.sessionFactory = entityManagerFactory.unwrap(SessionFactory.class);  
        this.transactionManager = new HibernateTransactionManager(this.sessionFactory);  
    }  
  
    @Override  
    public PlatformTransactionManager getTransactionManager() {  
        return this.transactionManager;  
    }  
}
```


다음으로는 리더를 구현하자. 이외 코드는 이쯤왔으면 어떻게 구성할지 알테니 생략하도록 하겠다.

```
@Bean  
@StepScope  
public HibernateCursorItemReader<Customer> customerItemReader(
EntityManagerFactory entityManagerFactory,  
@Value("#{jobParameters['city']}") String city) {  
    return new HibernateCursorItemReaderBuilder<Customer>()  
            .name("customerItemReader")  
            .sessionFactory(entityManagerFactory.unwrap(SessionFactory.class))  
            .queryString("from Customer where city = :city")  
            .parameterValues(Collections.singletonMap("city", city))  
            .build();  
}
```



##### 하이버네이트로 페이징 처리하기
---

하이버네이트는 배치 작업 시 페이징 기법도 지원한다.  `HibernatePagingItemReaderBuilder` 를 통해 `HibernatePagingItemReader` 를 생성할 수 있다. 


```
@Component  
public class SimpleHibernatePagingItemReader<T> {  
  
    @StepScope  
    public HibernatePagingItemReader<T> get(EntityManagerFactory entityManagerFactory, @Value("#{jobParameters['city']}") String city) {  
  
        return new HibernatePagingItemReaderBuilder<T>()  
                .name("customerItemReader")  
                .sessionFactory(entityManagerFactory.unwrap(SessionFactory.class))  
                .queryString("from Customer where city = :city")  
                .parameterValues(Collections.singletonMap("city", city))  
                .pageSize(10)  
                .build();  
  
    }  
}
```


실제로 애플리케이션을 실행해보면 아래처럼 `limit` 을 걸어 `paging` 사이즈를 지정한다.

```
Hibernate: 
    select
        customer0_.id as id1_0_,
        customer0_.address as address2_0_,
        customer0_.city as city3_0_,
        customer0_.first_name as first_na4_0_,
        customer0_.last_name as last_nam5_0_,
        customer0_.middle_initial as middle_i6_0_,
        customer0_.state as state7_0_,
        customer0_.zip_code as zip_code8_0_
    from
        customer customer0_
    where
        customer0_.city=? limit ?
```


앞서 예제에서는`queryString()` 을 통해 쿼리를 날렸다. 이외 하이버네이트에서 지원하는 API들도 알아보자.

```
queryName() : 하이버네이트 구성에 포함된 네임드 하이버네이트 쿼리를 참조
queryProvider() : 하이버네이트 쿼리를 프로그래밍으로 빌드하는 기능 HibernateQueryProvider 를 인자로 받음
nativeQuery() : 네이티브 SQL 쿼리를 실행한 뒤 결과를 하이버네이트로 매핑하는데 사용
```