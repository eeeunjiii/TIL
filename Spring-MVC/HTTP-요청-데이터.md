### HTTP 요청 데이터
- 클라이언트에서 서버로 HTTP 요청 메시지를 전달하는 방법에는 세 가지가 있다.
1. GET - 쿼리 파라미터
   - `/url**username=hello&age=20**`
   - 메시지 바디 없이 URL의 쿼리 파라미터에 데이터를 포함해서 전달한다.
   - 검색, 필터, 페이징 등에서 많이 사용하는 방식이다.
2. POST - HTML Form
   - content-type: application/x-www-form-urlencorded
   - 메시지 바디에 쿼리 파라미터 형식으로 전달한다. `username=hello&age=20`
   - 회원 가입, 상품 주문, HTML Form에서 주로 사용한다.
3. HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - 데이터 형식은 주로 JSON을 사용한다.
    - POST, PUT, PATCH
---
### GET 쿼리 파라미터

```java
@WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
public class RequestParamServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("[전체 파라미터 조회]");

        Enumeration<String> parameterNames=req.getParameterNames();
        
//        while(parameterNames.hasMoreElements()) {
//            String paramName = parameterNames.nextElement();
//            System.out.println(paramName + " = "+req.getParameter(paramName));
//        }
        
        req.getParameterNames().asIterator()
                .forEachRemaining(paramName -> System.out.println(paramName + " = "+req.getParameter(paramName)));
        System.out.println();

        System.out.println("[단일 파라미터 조회]");
        String username = req.getParameter("username");
        System.out.println("req.getParameter(user) = "+username);
        
        String age = req.getParameter("age");
        System.out.println("req.getParameter(age) = "+age);
        System.out.println();

        System.out.println("[이름이 같은 복수 파라미터 조회]");
        System.out.println("req.getParameterValues(username)");
        String[] usernames = req.getParameterValues("username");
        for(String name:usernames) {
            System.out.println("name = " + name);
        }
        
        resp.getWriter().write("OK");
    }
}
```
- message body 없이 URL의 쿼리 파라미터를 이용하여 서버에 데이터를 전달하도록 하였다.
- 쿼리 파라미터는 URL에 `?`를 시작으로 보낼 수 있다.
- `http://localhost:8080/request-param?username=hello&age=20`
- 해당 URL에 매핑되는 서블릿이 실행된다. 
- 동일한 이름을 가진 파라미터의 경우에 위의 코드처럼 해당 이름을 가진 값들을 모두 조회하지 않고 그 중 하나만 조회하도록 `getParameter`를 사용할 수 있다.
  - 그 중 제일 앞에 있는 값을 반환한다.