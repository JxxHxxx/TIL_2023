
`DriverManager` 는 항상 새로운 커넥션을 맺는다. `dataSourceDriverManager` 는 `DataSource` 인터페이스를 사용하기 위해 스프링에서 제공하는 클래스이다.

``DriverManager`` , `dataSourceDriverManager` 두 클래스가 어떻게 커넥션을 획득하는지 살펴보자.


*DriverManager*

```
@Test  
void driverManager() throws SQLException {  
    Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
    Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
}
```


*DataSourceDriverManager*

```
@Test  
void dataSourceDriverManager() throws SQLException {  
    DataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);  
    Connection con1 = dataSource.getConnection();  
    Connection con2 = dataSource.getConnection();  
}
```


큰 차이가 없어 보이지만 핵심은 **설정과 사용의 분리**다. `DriverManager` 는 커넥션을 획득할 때 설정과 사용을 함께한다. 반면  `DataSourceDriverManager`는 설정과 사용을 분리하였다.

애플리케이션 개발을 하면 보통 설정은 한 곳에서 하지만, 사용은 수 많은 곳에서 하게 된다. 이로 인해 변경이 일어났을 때 더 유연하게 대처할 수 있게 된다.
