
##### DDL AUTO 옵션

스프링 환경이라면 아래 옵션을 통해 DDL 생성 속성을 설정할 수 있다.

```
spring.jpa.hibernate.ddl-auto=create
```

해당 옵션에 지정할 수 있는 값과 설명이다.

```
create : 기존 테이블을 삭제하고 새로 생성한다.
create-drop : create 속성에 추가로 애플리케이션을 종료할 때 생성한 DDL 제거
update : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정
validate : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다. 이 설정은 DDL을 수정하지 않는다.
none : 자동 생성 기능을 사용하지 않는다.
```


*개발, 운영 환경 주의 사항*

```
개발 환경이라면 선호에 따라 DDL auto 옵션을 지정하면 된다.
예를 들어 자동화된 테스트를 진행하는 환경에서는 create, create-drop을 사용하면 적절할 것이다.

################################################################
운영 환경이라면 validate, none 옵션을 사용해야 한다. 이외 DDL이 수정될 수 있는 옵션을 선택한다면 치명적인 장애로 이어질 수 있다.
```



##### DDL 생성 기능


```
@Table(name = "jx_product",  
        indexes = @Index(name = "id_name", columnList = "name, id"),  
        uniqueConstraints = @UniqueConstraint(name = "unique_code",  columnNames = "code"))  
@Entity  
public class Product {  
  
    @Id  
    private String id;
    @Column(nullable = false, length = 10)  
    private String name;  
    private Integer quantity;  
    private String code;
```


`@Column` 의 `nullable` , `length` , `@UniqueConstraint` 애노테이션을 통해 필드 혹은 유니크 제약을 생성할 수 있다. 이런 기능들은 단지 DDL을 자동 생성할 때만 사용되고 JPA 실행 로직에는 영향을 주지 않는다. 따라서 스키마 자동 생성 기능을 사용하지 않고 직접 DDL을 만든다면 사용할 이유가 없다.