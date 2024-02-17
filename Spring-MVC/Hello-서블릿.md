### Hello 서블릿
- 스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 `@ServletComponentScan`을 지원한다.
- `@WebFilter`, `@WebServlet`, `@WebListener` 어노테이션들이 있는 클래스들을 등록한다.
- `ServletApplication`에 해당 어노테이션을 추가하면 된다.

```java
@ServletComponentScan
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}

}
```
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
    
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("HelloServlet.service");
        System.out.println("req = " + req);
        System.out.println("resp = " + resp);
        
        String username=req.getParameter("username");
        System.out.println("username = " + username);
        
        resp.setContentType("text/plain");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().write("hello "+username);
    }
}
```
- HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 `service` 메소드를 실행한다.
- 서버를 실행시키고 브라우저에 `http://localhost:8080/hello?username={username}` 을 입력한다.
- `req.getParameter(name)`는 `name`에 해당하는 파라미터 값을 반환한다. 값이 없을 경우에는 null을 반환한다.
---

### 서블릿 컨테이너 동작 방식
- 스프링 부트가 실행되면 내장 톰캣 서버를 생성한다.
- 내장 톰캣 서버는 서블릿 컨테이너에 서블릿들을 생성한다.
- HTTP 요청이 들어오면 웹 어플리케이션 서버는 HTTP 요청 메시지를 기반으로 request, response를 생성한다.
- 이를 기반으로 해당 URL과 매핑되는 서블릿을 실행한다. 서블릿의 실행이 종료되면 Response 객체 정보로 HTTP 응답을 생성한다.
- 참고로 HTTP 응답에서 Content-Length는 웹 어플리케이션 서버가 자동으로 생성해준다.
---

### welcome 페이지 추가
- webapp 경로에 index.html을 두면 `http://localhost:8080`를 호출하면 index.html 페이지가 열린다.

```html
<!DOCTYPE html>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<ul>
 <li><a href="basic.html">서블릿 basic</a></li>
</ul>
</body>
</html>
```
- `basic.html`은 다음과 같이 작성한다.
```html
<!DOCTYPE html>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<ul>
 <li>hello 서블릿
	 <ul>
		 <li><a href="/hello?username=servlet">hello 서블릿 호출</a></li>
	 </ul>
 </li>
 <li>HttpServletRequest
	 <ul>
		 <li><a href="/request-header">기본 사용법, Header 조회</a></li>
		 <li>HTTP 요청 메시지 바디 조회
			 <ul>
				 <li><a href="/request-param?username=hello&age=20">GET - 쿼리
파라미터</a></li>
				 <li><a href="/basic/hello-form.html">POST - HTML Form</a></li>
				 <li>HTTP API - MessageBody -> Postman 테스트</li>
			 </ul>
		 </li>
	 </ul>
 </li>
 <li>HttpServletResponse
	 <ul>
		 <li><a href="/response-header">기본 사용법, Header 조회</a></li>
		 <li>HTTP 응답 메시지 바디 조회
	 <ul>
		 <li><a href="/response-html">HTML 응답</a></li>
		 <li><a href="/response-json">HTTP API JSON 응답</a></li>
	 </ul>
	</li>
 </ul>
 </li>
</ul>
</body>
</html>
```