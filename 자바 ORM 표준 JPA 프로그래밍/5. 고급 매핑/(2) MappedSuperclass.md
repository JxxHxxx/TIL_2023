
상속 관계 매핑과 관련은 없다. 공통으로 사용되는 컬럼이 필요할 때 사용한다.

운영 환경에서 일반적으로 필요한 정보들이 있다. 예를 들어 
등록일, 수정일, 등록자, 수정자가 있다. 이 정보는 공통으로 필요한 경우가 많으니 이럴 때 활용할 수 있다.

- 참고 : `@Entity` 클래스는 엔티티나 `@MappedSuperclass` 로 지정한 클래스만 상속 가능


아래 같이 지정하고 필요한 엔티티에서 상속받아 사용하면 된다.

```
@MappedSuperclass  
public abstract class BaseEntity {  
  
    private String createBy;  
    private LocalDateTime createTime;  
    private String modifiedBy;  
    private LocalDateTime modifiedTime;  
  
    public BaseEntity() {  
        this.createTime = LocalDateTime.now();  
    }  
}
```


```
@Entity
public abstract class Item extends BaseEntity {
```