
#### JAR
자바는 여러 클래스와 리소스를 묶어서 `JAR` (Java Archive)라고 하는 압축 파일을 만들 수 있다. 이 파일은 JVM위에서 직접 실행되거나 또는 다른 곳에서 사용하는 라이브러리로 제공된다.

직접 실행하는 경우 `main()` 메서드가 필요하고, `MANIFEST.MF` 파일에 실행할 메인 메서드가 있는 클래스를 지정해두어야 한다.


#### WAR
WAR(Web Application Archive)로 웹 애플리케이션 서버에 배포할 때 사용하는 파일이다. WAR는 웹 애플리케이션 서버 위에서 실행된다. 웹 애플리케이션 서버 위에서 실행되고, HTML 같은 정적 리소스와 클래스 파일을 모두 함께 포함하기 때문에 JAR와 비교해서 구조가 더 복잡하다. **그리고 WAR 구조를 지켜야 한다.**


*WAR 구조*

![[Pasted image 20230905000759.png]](https://github.com/JxxHxxx/TIL_2023/blob/master/%EC%98%81%ED%95%9C%EB%8B%98%20%EA%B0%95%EC%9D%98/%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20-%20%ED%95%B5%EC%8B%AC%20%EC%9B%90%EB%A6%AC%EC%99%80%20%ED%99%9C%EC%9A%A9/images/Pasted%20image%2020230905000759.png)

- `WEB-INF`
	- `classes` 실행 클래스 모음
	- `lib` 라이브러리 모음
	- `web.xml` 웹 서버 배치 설정 파일(생략 가능)
- `index.html` 정적 리소스

`WEB-INF` 폴더 하위는 자바 클래스와 라이브러리, 설정 정보가 들어가는 곳이다. `WEB-INF`를 제외한 나머지 영역은 `HTML` ,`CSS` 같은 정적 리소스가 사용되는 영역이다.
	
