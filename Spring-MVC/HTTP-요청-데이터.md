### HTTP 요청 데이터
- 클라이언트에서 서버로 HTTP 요청 메시지를 전달하는 방법에는 세 가지가 있다.
1. GET - 쿼리 파라미터
   - `/url?username=hello&age=20`
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
---
### HTTP 요청 데이터 - POST HTML Form
- 해당 방식은 주로 회원 가입, 상품 주문 등에서 사용하는 방식이다.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-param" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
 ```
- 위의 코드를 그대로 사용하여 주소창에 `http://localhost:8080/request-param` 을 입력하면 된다.
- POST의 HTML Form을 전송하면 웹 브라우저는 다음과 같은 형식으로 HTTP 메시지를 만든다.
   - 요청 URL: `http://localhost:8080/request-param`
   - content-type: `application/x-www-form-urlencoded`
   - message body: `username=hello&age=20`
- 앞서 살펴본 GET 방식의 쿼리 파라미터와 동일한 결과인 것을 확인할 수 있다.
- 웹 브라우저 입장에서는 두 방식에 차이가 있지만, 서버 측에서는 동일한 방식으로 처리하기 때문에 `req.getParameter()`를 편리하게 구분 없이 사용할 수 있다.
---
### HTTP 요청 데이터 - API 메시지 바디 (단순 텍스트)
- 이 경우 HTTP 메시지 바디의 데이터를 InputStream을 사용해서 읽을 수 있다.
```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        System.out.println("messageBody = " + messageBody);
        resp.getWriter().write("ok");
    }
}
```
- 이때 `inputStream`은 byte 코드를 반환하기 때문에 읽을 수 있도록 문자표를 지정해줘야 한다. → `StandardCharsets.UTF_8`
- HTTP 메시지 결과를 보면 다음과 같다.
   - POST `http://localhost:8080/request-body-string`
   - content-type: `text/plain`
   - message body: `hello`
  

### HTTP 요청 데이터 - API 메시지 바디 (JSON)
- JSON 형식을 이용한 데이터 전달을 보기 전에 JSON 형식으로 파싱할 수 있는 객체를 생성해야 한다.
```java
package hello.servlet.basic;

@Getter
@Setter
public class HelloData {
    private String username;
    private int age;
}

```
```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper(); 
    
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletInputStream inputStream = req.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        System.out.println("helloData.getUsername() = " + helloData.getUsername());
        System.out.println("helloData.getAge() = " + helloData.getAge());
        
        resp.getWriter().write("ok");
    }
}
```
- 위의 코드 중에서 `ObjectMapper`는 JSON 형식을 사용할 때, **응답들을 직렬화(Object -> JSON)** 하고 **요청들을 역직렬화(JSON -> Object)** 할 때 사용한다.
- HTTP 메시지 결과를 보면 다음과 같다.
   - POST `http://localhost:8080/request-body-json`
   - content-type: `application/json`
   - message body: `{"username": "hello", "age": 20}`