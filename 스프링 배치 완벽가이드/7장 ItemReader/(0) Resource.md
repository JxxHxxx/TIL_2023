
`Resource` 는 다양한 리소스(`txt`, `json` , `xml` 등..)에 대한 접근을 추상화한 인터페이스이다. 즉 `Resource` 를 통해 리소스에 접근할 수 있다. 기본적으로 `ClassPathContextResource` 구현체를 주입해서 사용한다.

자세히 알 필요는 없지만 기본적으로 스프링 부트에서는 다폴트 `path` 로  `main/resources` 잡는다.

예를 들어 아래 `customer.json` 에 접근하기 위해서는 클래스 패스에 `/customer.json` 를 명시하면 된다.

![[Pasted image 20231119174958.png]](images/Pasted%20image%2020231119174958.png)

