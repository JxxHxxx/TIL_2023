

간단하게 서블릿 클래스를 하나 구현해두자.

```
@WebServlet(urlPatterns = "/test")  
public class TestServlet extends HttpServlet {  
  
    @Override  
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        System.out.println("TestServlet.service");  
        super.service(req, resp);  
    }  
}
```


이후 프로젝트가 있는 루트 디렉토리로 이동해서 빌드를 수행하자.
`window` 기준 `gradlew build`

![[Pasted image 20230904235651.png]]


`build/libs` 경로로 들어가면 `war` 파일이 생성된걸 볼 수 있다.

![[Pasted image 20230904235903.png]]


압축을 풀면 우리가 작성한 컴파일된 클래스 파일이 존재하는걸 볼 수 있다.

![[Pasted image 20230905000027.png]]

이 `war` 파일을 톰캣 내부 `webapps` 디렉토리에 넣어주면 된다. 그리고 강의에서는 `war` 파일명을 ROOT로 변경한다. 아마도 특정 설정을 변경하지 않으면 ROOT 라는 `war` 파일을 인식하게 만들어두었나보다.

이 상태에서 톰캣을 실행하면 `war` 파일 압축을 푼다. 그래서 아래처럼 ROOT 디렉터리가 만들어지게 된다.

![[Pasted image 20230905001642.png]]

