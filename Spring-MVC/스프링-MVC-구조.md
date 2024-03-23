### 스프링 MVC 구조
- 이전에 직접 만들었던 MVC 프레임워크에서 Front Controller는 실제 Spring MVC에서의 `DispatcherServlet`과 동일하다.
- 프론트 컨트롤러는 하위 경로에 들어오는 모든 요청들을 받았는데, `DispatcherServlet`도 동일하다.
  - 스프링 부트는 `DispatcherServlet`을 서블릿으로 자동 등록하면서 모든 경로에 대해서 매핑한다.
- 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()`가 호출된다. (`@Override`)
- `FrameworkServlet.service()`를 호출하면서 여러 메소드가 호출되면서 `Dispatcher.doDispatch()`가 호출된다.


**`DispatchServlet**
```java
public class DispatcherServlet extends FrameworkServlet {
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception{
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        // 핸들러 조회
        mappedHandler = getHandler(processedRequest);
        if (mappedHandler == null) { // 매핑되는 핸들러가 없을 경우
            noHandlerFound(processedRequest, response); // 핸들러 조회 실패
            return;
        }
        
        // 핸들러 조회 성공 시 핸들러를 처리할 수 있는 어댑터 조회
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

        // 조회된 어댑터 실행 
        mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                         @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                         @Nullable Exception exception) throws Exception {

        // Did the handler return a view to render?
        if (mv != null && !mv.wasCleared()) {
            render(mv, request, response); // view render
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        }
    }
}
```
1. 핸들러 조회
2. 핸들러 어댑터 조회
3. 핸들러 어댑터 실행
4. 핸들러 어댑터의 핸들러 실행
5. ModelAndView 반환
6. viewResolver 호출
7. View 반환
8. 뷰 렌더링