### Model 추가 - V3
- V2의 컨트롤러들을 보면 `HttpServletRequest`, `HttpServletResponse`를 사용하지 않는 경우가 있다.
- 요청 파라미터의 정보는 자바의 Map으로 넘기도록 하면 지금의 구조에서는 컨트롤러가 서블릿 기술을 몰라도 동작할 수 있다.
- `request` 객체를 Model로 사용하는 대신에 **별도의 Model 객체를 만들어서 반환**하면 된다.
- 또한, V2에서 `viewPath`를 보면 `/WEB-INF/view/members`가 반복된다.
- 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 할 수 있다.


1. 클라이언트에서 HTTP 요청을 보내면
2. 프론트 컨트롤러에서는 매핑 정보를 확인하여 요청이 들어온 URI와 매핑되는 컨트롤러를 호출한다.
3. 해당 컨트롤러는 로직을 수행하고 `ModelView`를 프론트 컨트롤러에 반환한다.
4. 프론트 컨트롤러에서는 **viewResolver**를 호출하여 실제 물리 위치를 갖는 `MyView`를 프론트 컨트롤러에 반환한다.
5. 프론트 컨트롤러는 이전과 동일하게 `MyView`의 `render()`를 호출하여 JSP로 forward한다.


**ModelView**
```java
public class ModelView {
    private String viewName;
    private Map<String, Object> model=new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```
- 이전까지는 서블릿에 종속적인 `HttpServlerRequest`를 사용했었다.
- Model도 `req.setAttribute()`를 통해 데이터를 저장해 View에 전달했다.
- 이를 해결하기 위해 Model을 직접 만들고, `viewName`도 전달하는 객체를 생성했다.
- View를 렌더링할 때 필요한 model 객체를 Map의 형태로 갖고 있다.


**ControllerV3**
```java
public interface ControllerV3 {
    ModelView process(Map<String, String> paramMap);
}

```
- 각 컨트롤러들은 `MyView`가 아니라 `ModelView`를 반환한다.


**회원 등록 컨트롤러**
```java
public class MemberFormControllerV3 implements ControllerV3 {

    @Override
    public ModelView process(Map<String, String> paramMap) {
        return new ModelView("new-form");
    }
}
```

**회원 저장 컨트롤러**
```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository=MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age=Integer.parseInt(paramMap.get("age"));

        Member member=new Member(username, age);
        memberRepository.save(member);

        ModelView mv=new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```
- View에 전달해야 할 데이터 (모델)가 있다면 Map에 저장한다.
- 파라미터들은 Map의 형태로 전달받는다.

**회원 조회 컨트롤러**
```java
public class MemberListControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository=MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        List<Member> members=memberRepository.findAll();
        ModelView mv=new ModelView("members");
        mv.getModel().put("members", members);
        return mv;
    }
}
```

**프론트 컨트롤러**
```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {
    private Map<String, ControllerV3> controllerMap=new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestURI=req.getRequestURI();

        ControllerV3 controller=controllerMap.get(requestURI);
        if(controller==null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // paramMap
        Map<String, String> paramMap=createParamMap(req);
        ModelView mv=controller.process(paramMap);
        
        String viewName=mv.getViewName(); // 논리이름을 얻을 수 있음 ex) new-form
        MyView view= viewResolver(viewName); // 논리 이름 -> 물리적 이름
        
        view.render(mv.getModel(), req, resp);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest req) {
        Map<String, String> paramMap=new HashMap<>();
        req.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, req.getParameter(paramName)));
        return paramMap;
    }
}
```
- 요청 파라미터들은 `HttpServletRequest`에서 꺼내와 Map의 형태로 저장한다.
- 이 요청 파라미터 map은 각 컨트롤러에 전달되고 컨트롤러들은 필요한 model들을 map에서 꺼내서 사용한다.
- 로직 수행을 마친 컨트롤러는 `ModelView` 객체를 반환한다. 여기서 `viewName`을 추출하여
- 이를 `viewResolver`에 넘겨 `MyView`를 반환 받는다.
- 이때 `MyView` 객체의 `render()`를 호출할 때 model도 같이 전달한다.
- JSP는 `req.getAttribute()`로 데이터를 조회하기 때문에 model의 데이터를 꺼내서 `req.setAttribute()`로 변환해서 담아둔다.


**수정된 MyView**
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

    public void render(Map<String, Object> model, HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        modelToRequestAttribute(model, req);
        RequestDispatcher dispatcher=req.getRequestDispatcher(viewPath); // jsp로 forward
        dispatcher.forward(req, resp);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest req) {
        
        // jsp에서 setAttribute를 보고 값을 가져온다.
        model.forEach((key, value) -> req.setAttribute(key, value)); // 필요한 모든 파라미터를 꺼내서 attribute 설정
    }
}
```