### `HttpServletRequest`
- 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다. 그 결과를 HttpServletRequest 객체에 담아서 제공한다.
- START_LINE
    - HTTP 메소드
    - URL
    - 쿼리 스트링
    - 스키마, 프로토콜
- HEADER
    - 헤더 조회
- BODY
    - form 파라미터 형식 조회
    - message body 데이터 직접 조회


- 이외에도 `HttpServletRequest` 객체는 추가로 여러 부가기능을 제공한다.


**임시 저장소 기능**
- 한 HTTP 요청이 시작될 때부터 끝날 때까지 유지되는 임시 저장소 기능
    - 저장: `req.setAttribute(name, value)`
    - 조회: `req.getAttribute(name)`
  

- `HttpServletRequest`가 제공하는 기본 기능들을 코드를 통해 확인할 수 있다.

```java
@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        printStartLIne(req);

        resp.getWriter().write("ok");
    }
}
```
```java
private void printStartLIne(HttpServletRequest req) {
    System.out.println("--- REQUEST-LINE - start ---");

    System.out.println("req.getMethod = "+req.getMethod()); // GET
    System.out.println("req.getProtocol = "+req.getProtocol()); // HTTP/1.1
    System.out.println("req.getScheme = "+req.getScheme()); // http
    System.out.println("req.getRequestURL = "+req.getRequestURL()); // http://localhost:8080/request=header
    System.out.println("req.getRequestURI = "+req.getRequestURI()); // /reqeust-header
    System.out.println("req.getQueryString = "+req.getQueryString()); // username={username}
    System.out.println("req.isSecure = "+req.isSecure()); // https 사용 유무
    System.out.println("--- REQUEST-LINE - end ---");
    System.out.println();
}
```
```java
private void printHeader(HttpServletRequest req) {
    System.out.println("--- HEADER-LINE - start ---");

    Enumeration<String> headerNames = req.getHeaderNames();
    while(headerNames.hasMoreElements()) {
      String headerName = headerNames.nextElement();
      System.out.println("headerName = " + req.getHeader(headerName));
    }

    req.getHeaderNames().asIterator()
            .forEachRemaining(headerName -> System.out.println("headerName = " + req.getHeader(headerName)));

    System.out.println("--- HEADER-LINE - end ---");
    System.out.println();
}
```