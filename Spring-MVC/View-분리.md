### View 분리 - V2
- 이전의 프론트 컨트롤러 부분을 보면 컨트롤러에서 뷰로 이동하는, 다음 코드가 공통적으로 있다.

```java
 String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = req.getRequestDispatcher(viewPath);
        dispatcher.forward(req, resp);
```

- 이렇게 반복되는 부분을 분리해야 한다.
1. 클라이언트에서 HTTP 요청을 보내면
2. 프론트 컨트롤러에서는 매핑 정보를 확인하여 요청이 들어온 URI와 매핑되는 컨트롤러를 호출한다.
3. 해당 컨트롤러에서 로직을 수행하고 마지막에 View를 프론트 컨트롤러에 반환한다.
4. 프론트 컨트롤러에서는 `render()` 함수를 호출하고 view에서는 JSP를 forward하게 한다.

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        RequestDispatcher dispatcher=req.getRequestDispatcher(viewPath);
        dispatcher.forward(req, resp);
    }
}
```
- 앞의 코드와 동일한 로직을 수행하는 `MyView` 클래스를 생성하였다.


```java
public interface ControllerV2 {
    
    MyView process(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException;
    
}
```
- 컨트롤러가 View를 반환하도록 하였다.

**회원 등록 컨트롤러**
```java
public class MemberFormControllerV2 implements ControllerV2 {

    @Override
    public MyView process(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

**회원 저장 컨트롤러**
```java
public class MemberSaveControllerV2 implements ControllerV2 {
    
    private final MemberRepository memberRepository=MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        int age=Integer.parseInt(req.getParameter("age"));
        
        Member member=new Member(username, age);
        memberRepository.save(member);
        
        req.setAttribute("member", member);
        
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```

**회원 조회 컨트롤러**
```java
public class MemberListControllerV2 implements ControllerV2 {

    private final MemberRepository memberRepository=MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        req.setAttribute("members", members);
        return new MyView("/WEB-INF/views/members.jsp");
    }
}
```

- 각 컨트롤러에서 로직을 수행하고 `MyView` 객체에 `viewPath`를 넣어서 프론트 컨트롤러에 반환한다.


**프론트 컨트롤러**
```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private final Map<String, ControllerV2> controllerMap=new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestURI=req.getRequestURI();

        ControllerV2 controller= controllerMap.get(requestURI);

        if(controller==null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyView view=controller.process(req, resp);
        view.render(req, resp);
    }
}
```
- 이전 프론트 컨트롤러와 동일하지만 각 컨트롤러에서 `MyView`를 반환하면 프론트 컨트롤러에서 `render()`를 실행한다.
- 이렇게 하면 `render()`를 호출하는 부분을 일관되게 처리할 수 있다.