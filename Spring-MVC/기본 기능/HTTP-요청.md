### HTTP 요청 - 기본, 헤더 조회
```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest req,
                          HttpServletResponse resp,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie) {
        log.info("req: {}", req);
        log.info("resp: {}", resp);
        log.info("httpMethod: {}", httpMethod);
        log.info("locale: {}", locale);
        log.info("headerMap: {}", headerMap);
        log.info("header host: {}", host);
        log.info("myCookie: {}", cookie);
        
        return "ok";
    }
}
```
- `@RequestHeader`는 모든 HTTP 헤더를 `MultiValueMap`의 형식으로 조회한다.
  - 하나의 키에 여러 값을 받을 수 있다.
- `@RequestHeader("host") String host`는 특정 HTTP 헤더를 조회한다.
- `@CookieValue(value = "myCookie", required = false) String cookie`는 특정 쿠키를 조회한다.
---

### HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

**HTTP 요청 데이터**
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


- `HttpServletRequest`의 `req.getParameter()`를 사용하면 다음의 두 가지 요청 파라미터를 조회할 수 있다.
  - **GET, 쿼리 파라미터 전송**
  - **POST, HTML Form 전송**
  - 어떤 방식이든 둘 다 형식이 같기 때문에 구분없이 조회할 수 있다.

```java
@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String username = req.getParameter("username");
        int age = Integer.parseInt(req.getParameter("age"));
        log.info("username= {}, age= {}", username, age);

        resp.getWriter().write("ok"); // @RestController 대신 사용
    }
}
```
- `http://localhost:8080/request-param-v1?username=hello&age=20` 을 실행하면 `username`과 `age`의 값을 얻을 수 있다.
- 다음은 POST, HTML Form 전송에 대한 HTML이다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-param-v1" method="post">
    username: <input type="text" name="username" />
    age:      <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```
- 태그 안에 지정한 `name`과 매핑된다.
---

### HTTP 요청 파라미터 - @RequestParam
- `@RequestParam`을 사용하면 `HttpServletRequest`를 사용하지 않고 편리하게 파라미터를 조회할 수 있다.

```java
@Slf4j
@Controller
public class RequestParamController {
    @ResponseBody
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge) {

        log.info("username= {}, age= {}", memberName, memberAge);
        return "ok";
    }
}
```
- `@RequestParam`의 name 속성이 파라미터 이름으로 사용된다. ("username", "age")
- HTTP 파라미터 이름이 변수 이름과 동일하면 `name` 속성을 생략할 수 있다.
  - `@RequestParam String username, @RequestParam int age`
- `String`, `int`, `Integer` 등의 단순 타입이면 `@RequestParam` 자체를 생략할 수 있다.
- 애노테이션에 이름을 생략하지 않고 이름을 항상 같이 적는 것을 권장한다.


**파라미터 필수 여부 (required)**
```java
@Slf4j
@Controller
public class RequestParamController {
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required=true) String username,
            @RequestParam(required = false) Integer age) { // 이 경우 age에 값을 넣지 않으면 null이 들어가는데 primitive 타입은 null X -> Integer로 변경
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
}
```
- 위와 같이 `required`에 `true`로 지정하면 해당 파라미터에는 값이 꼭 있어야 한다
- `username`이 없다면 400 예외가 발생한다.
- 이때 파라미터 이름만 있고 값이 없다면 빈 문자로 통과한다. (`/request-param-required?username=`)
- 위의 경우 `age`는 기본형 (primitive)이기 때문에 null을 입력할 수 없다. 따라서 `Integer`로 변경하거나 `defaultValue`를 사용해야 한다.


**기본값 적용 (defaultValue)**
```java
@Slf4j
@Controller
public class RequestParamController {
    @ResponseBody
    @RequestMapping("/request-param-default")
    public String requestParamDefaultValue(
            @RequestParam(required=true, defaultValue = "hello") String username,
            @RequestParam(required = false, defaultValue = "-1") Integer age) { // default value를 쓰면 required를 굳이 쓰지 않아도 됨, 빈 문자도 처리해줌
        log.info("username= {}, age= {}", username, age);
        return "ok";
    }
}
```
- 파라미터에 값이 없는 경우 `defaultValue`에 지정되어 있는 값이 적용된다.
- 따라서 `required` 속성이 없어도 된다.


**파라미터를 Map으로 조회하기**
```java
@Slf4j
@Controller
public class RequestParamController {
    @ResponseBody
    @RequestMapping("/request-param-map")
    public String requestParamMap(
            @RequestParam Map<String, Object> paramMap) {
        log.info("username= {}, age= {}", paramMap.get("username"), paramMap.get("age"));
        return "ok";
    }
}
```
- 파라미터의 값이 1개인 것이 확실하다면 `Map`을 사용해도 되지만 그렇지 않다면 `MultiValueMap`을 사용해야 한다.
