### MVC 패턴 (개요)
- 하나의 서블릿이나 JSP에 비즈니스 로직과 뷰 렌더링까지 모두 처리하면 너무 많은 역할을 맡게 된다.
- 결과적으로 유지보수가 어려워진다. 비즈니스 로직을 호출하는 부분에 변경이 발생하면 해당 코드를 수정해야 한다.


- 비즈니스 로직과 UI를 수정하는 변경의 라이프 사이클이 다르다는 문제점이 있다.
- 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수하기 좋지 않다. (UI가 많이 변경되면 함께 변경될 가능성도 있다.)


- 또한, JSP 같은 뷰 템플릿은 화면을 렌더링하는 데 최적화되어 있기 때문에 이 부분의 역할만 담당하는 것이 좋다.


- MVC 패턴은 컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 구분한 것이다.
- 이때 컨트롤러는 HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
- 모델은 뷰에 출력할 데이터를 담아둔다. 뷰에 필요한 데이터를 모두 모델에 담아서 전달하기 때문에 뷰는 비즈니스 로직이나 데이터 접근을 신경쓰지 않아도 된다.
- 뷰는 모델에 담겨 있는 데이터를 사용해서 화면을 그리는 일을 한다.
---
### MVC 패턴 (적용)
**회원 등록 폼 - 컨트롤러**

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String viewPath="/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
        dispatcher.forward(req, resp); 
    }
}
```
- 해당 코드에서 `dispatcher.forward()`는 다른 서블릿이나 JSP로 이동할 수 있는 기능이다.
- 다른 리소스를 호출할 때 서버 내부에서 다시 호출이 발생한다.
- redirect와 같이 응답이 가고 클라이언트로부터 다시 요청이 발생하는 것이 아니기 때문에 URL의 변경이 일어나지 않는다
- `viewPath`에서 `WEB-INF`의 경로 안에 JSP가 있으면 외부에서 직접 호출할 수 없다. 항상 컨트롤러를 통해서 호출된다.


**회원 등록 폼 - 뷰**
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username" />
    age:      <input type="text" name="age" />
 <button type="submit">전송</button>
</form>
</body>
</html>
```
- 추가로 `action`을 보면 `/`를 붙이지 않고 `save`로 사용했다. 이는 상대경로인 것을 의미한다. 이와 같이 사용하면 폼을 전송할 때 현재 URL이 속한 계층 경로 + `save`가 된다. → `/servlet-mvc/members/save`


**회원 저장 - 컨트롤러**
```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String username=req.getParameter("username");
        int age=Integer.parseInt(req.getParameter("age"));

        Member member=new Member(username, age);
        memberRepository.save(member);

        // Model 에 데이터를 보관해야 한다.
        req.setAttribute("member", member); // 내부 저장소에 저장

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
        dispatcher.forward(req, resp);
    }
}
```
- `HttpServletRequest`를 Model로 사용한다. (Map의 형태)
- `req.setAttribute()`: request 객체에 데이터를 보관해서 뷰에 전달할 수 있다.
- `req.getAttribute()`: 보관된 데이터를 꺼낸다.


**회원 저장 - 뷰**
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
	 <meta charset="UTF-8">
</head>
<body>
성공
<ul>
	 <li>id=${member.id}</li>
	 <li>username=${member.username}</li>
	 <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```
- 이전에 한 것처럼 `<%=req.getAttribute("member")%>` 로 모델에 저장한 member 객체를 꺼낼 수 있지만 복잡하다.
- JSP에서는 `${   }` 문법을 제공한다. request의 attribute에 담긴 데이터를 쉽게 조회할 수 있다.
- `${   }` 안에 있는 `member`는 앞서 지정한 Attribute의 키 값과 매핑된다.