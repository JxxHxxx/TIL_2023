6 - 3 에서는 두 가지 상이한 레코드 포맷을 상호 관련이 없는 두 아이템으로 처리하는 방법을 살펴봤다.

하지만 레코드를 유심히 살펴보면 실제로 레코드 사이에 관계가 있음을 알 수 있다.

```
CUST,Warren,Q,Darrow,8272,4th Street,New York,IL,76091  
TRANS,1165965,2011-01-22 00:13:29,51.43  
CUST,Ann,V,Gates,9247,Infinite Loop Drive,Hollywood,NE,37612  
CUST,Erica,I,Jobs,8875,Farnam Street,Aurora,IL,36314  
TRANS,8116369,2011-01-21 20:40:52,-14.83  
TRANS,8116369,2011-01-21 15:50:17,-45.45  
TRANS,8116369,2011-01-21 16:52:46,-74.6  
TRANS,8116369,2011-01-22 13:51:05,48.55  
TRANS,8116369,2011-01-22 16:51:59,98.53
```

고객 레코드가 나온 뒤, 해당 고객의 N개의 거래 내역 레코드가 나온는 구조이다. 이렇게 관계가 있는 레코드를 처리하려면 어떻게 해야 할까?

현재는 `FlatFileItemReader` 로는 어디까지가 레코드의 끝인지를 판단할 수 없다. (고객 레코드 + 해당 고객의 거래 내역 레코드) 이를 처리하기 위해서 새로운 `ItemReader` 를 구현한다.

다음은 이를 구현한 코드 전문이다. 설명은 하단에 존재한다.

```
@Slf4j  
@Component  
public class CustomerFileReaderV4 implements ItemStreamReader<CustomerV4> {  
  
    private Object curItem = null;  
    private ItemStreamReader<Object> delegate;  
  
    public CustomerFileReaderV4(ItemStreamReader<Object> delegate) {  
        this.delegate = delegate;  
    }  
  
    @Override  
    public CustomerV4 read() throws Exception {  
        if (curItem == null) {  
            curItem = delegate.read();  
            if (curItem != null) {  
                log.info("curItem {} , class {}", curItem.toString(), curItem.getClass());  
            }  
        }  
  
        CustomerV4 item = (CustomerV4) curItem;  
        curItem = null;  
  
        if (item != null) {  
            item.setTransactions(new ArrayList<>());  
  
            while (peek() instanceof Transaction) {  
                item.getTransactions().add((Transaction) curItem);  
                if (curItem != null) {  
                    log.info("call Transaction record {} , ", curItem.toString());  
                }  
                curItem = null;  
            }  
        }  
        return item;  
    }  
  
    private Object peek() throws Exception {  
        if (curItem == null) {  
            curItem = delegate.read();  
        }  
        return curItem;  
    }  
  
    @Override  
    public void open(ExecutionContext executionContext) throws ItemStreamException {  
        delegate.open(executionContext);  
    }  
  
    @Override  
    public void update(ExecutionContext executionContext) throws ItemStreamException {  
        delegate.update(executionContext);  
    }  
  
    @Override  
    public void close() throws ItemStreamException {  
        delegate.close();  
    }  
}

// Configration 클래스

@Bean  
public Step copyFileStep() {  
	log.info("start step");  
	return stepBuilderFactory.get("copyFileStep")  
			.<CustomerV4, CustomerV4>chunk(10)  
			.reader(customerFileReader())  // FlatFileItemReader 를 래핑한 객체가 등록됨
			.writer(itemWriter())  
			.build();  
}  

@Bean  
@StepScope    
public FlatFileItemReader customerItemReader(@Value("#{jobParameters['customerFile']}") Resource inputFile) {  
	return new FlatFileItemReaderBuilder<CustomerV4>()  
			.name("customerItemReader")  
			.resource(inputFile)  
			.lineMapper(lineMapper())  
			.build();  
}

@Bean  
public CustomerFileReaderV4 customerFileReader() {  
    return new CustomerFileReaderV4(customerItemReader(null));  
}
```


(1) : `ItemReader` 대신 `ItemStreamReader` 를 구현했다.

`ItemStreamReader` 는 `read()` 이외에도 읽을 대상 리소스를 열고 닫는 것을 제어하는 메서드를 구현해야 한다.  `CustomerFileReaderV4` 클래스는 단순히 뒷단의 `FlatFileItemReader` 를 래핑할 뿐이다. 즉, 리소스 관리는 `FlatFileItemReader` 에서 수행하기 때문에 `ItemStreamReader` 를 통해 이를 위임해주어야 한다.

만약 `CustomerFileReaderV4` 를 `ItemReader` 로 구현하면 아래와 같이 리소스를 열지 않았다는 메시지와 함께 예외가 발생한다.

![[Pasted image 20231211152336.png]]

(2) : 래핑한 이유

이건 내피셜이다. `FlatFileItemReader` 는 온전히 책임을 다하고 있다. 여기에 기능을 추가하는 건 해당 클래스가 너무 많은 책임을 진다고 판단했다고 본다. 이로 인해 신규 기능은 `CustomerFileReaderV4` 를 구현하고 위임 패턴을 적용한 것이다.