
`JpaItemWriter` 를 보면 JPA의 `EntityManager` 를 감싼 간단한 래퍼에 불과하다. 청크가 완료되면 청크 내의 아이템 목록이 `JpaItemWriter` 로 전달된다. `JpaItemWriter` 는 모든 아이템을 저장한 뒤 `flush` 를 호출하기 전에 아이템 목록 내 아이템을 순회하면서 아이템마다 `EntityManager` 의 `merge` 메서드를 호출한다.

즉, `Detached` 된 엔티티를 `merge` 메서드를 통해 영속화한 다음 마지막에 `flush` 를 호출하여 DB에 반영한다.


이제 `JPA` 를 잡에 연결해야 한다. 다음 두 가지 작업이 필요하다.

1. `JpaTransactionManager` 를 생성하는 `BatchConfigurer` 구현체를 작성한다.
2. `JpaItemWriter` 를 구성한다.


먼저  `BatchConfigurer` 구현체인 `JpaBatchConfigurer` 를 구현한 코드이다. 

```
@Component  
public class JpaBatchConfigurer implements BatchConfigurer {  
  
    private DataSource dataSource;  
    private EntityManagerFactory entityManagerFactory;  
    private JobRepository jobRepository;  
    private PlatformTransactionManager platformTransactionManager;  
    private JobLauncher jobLauncher;  
    private JobExplorer jobExplorer;  
  
    public JpaBatchConfigurer(DataSource dataSource, EntityManagerFactory entityManagerFactory) {  
        this.dataSource = dataSource;  
        this.entityManagerFactory = entityManagerFactory;  
    }  
  
    @Override  
    public JobRepository getJobRepository() throws Exception {  
        return this.jobRepository;  
    }  
  
    @Override  
    public PlatformTransactionManager getTransactionManager() throws Exception {  
        return this.platformTransactionManager;  
    }  
  
    @Override  
    public JobLauncher getJobLauncher() throws Exception {  
        return this.jobLauncher;  
    }  
  
    @Override  
    public JobExplorer getJobExplorer() throws Exception {  
        return this.jobExplorer;  
    }  
  
    @PostConstruct  
    public void initialize() throws Exception {  
        try {  
            JpaTransactionManager transactionManager = new JpaTransactionManager(entityManagerFactory);  
            transactionManager.afterPropertiesSet();  
  
            this.platformTransactionManager = transactionManager;  
  
            this.jobRepository = createJobRepository();  
            this.jobLauncher = createJobLauncher();  
            this.jobExplorer = createJobExplorer();  
  
        } catch (Exception e) {  
            throw new BatchConfigurationException(e);  
        }  
    }  
  
    private JobLauncher createJobLauncher() throws Exception {  
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();  
        jobLauncher.setJobRepository(this.jobRepository);  
        jobLauncher.afterPropertiesSet();  
        return jobLauncher;  
    }  
  
    private JobExplorer createJobExplorer() throws Exception {  
        JobExplorerFactoryBean jobExplorerFactoryBean = new JobExplorerFactoryBean();  
        jobExplorerFactoryBean.setDataSource(this.dataSource);  
        jobExplorerFactoryBean.afterPropertiesSet();  
        return jobExplorerFactoryBean.getObject();  
    }  
  
    private JobRepository createJobRepository() throws Exception {  
        JobRepositoryFactoryBean jobRepositoryFactoryBean = new JobRepositoryFactoryBean();  
        jobRepositoryFactoryBean.setDataSource(this.dataSource);  
        jobRepositoryFactoryBean.setTransactionManager(this.platformTransactionManager);  
        jobRepositoryFactoryBean.afterPropertiesSet();  
  
        return jobRepositoryFactoryBean.getObject();  
    }  
}
```


다음으로는 Job 을 구현하는 코드 중 변경된 부분이다. 생각보다 복잡하지 않다. `reader` ~ `itemprocessor` 로부터 만들어진 청크를 받아 쓰기 작업을 진행한다.

```
@Bean  
@StepScope  
public JpaItemWriter<WriteCustomerV2> jpaItemWriter(EntityManagerFactory entityManagerFactory) {  
    JpaItemWriter<WriteCustomerV2> jpaItemWriter = new JpaItemWriter<>();  
    jpaItemWriter.setEntityManagerFactory(entityManagerFactory);  
    return jpaItemWriter;  
}
```