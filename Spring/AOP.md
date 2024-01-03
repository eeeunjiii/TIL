# AOP

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
//@Component -> 해당 어노테이션을 붙이면 자동으로 스프링에서 빈으로 등록해줌
public class TimeTraceAop {

		@Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start=System.currentTimeMillis();
        System.out.println("start: "+joinPoint.toString());

        try{
            return joinPoint.proceed();
        } finally{
            long end=System.currentTimeMillis();
            long time=end-start;
            System.out.println("end: "+joinPoint.toString()+" "+time+" ms");
        }
    }
}
```

- 모든 메소드에 메소드의 실행 시간을 계산하는 로직을 추가하는 것은 번거로운 일이다. 핵심 기능이 아닌(주요 기능이 아닌) 공통 관심사에 대한 로직이 필요한 경우에는 AOP를 활용한다.
- AOP를 사용하려면 스프링 빈으로 등록해야 한다. `@Component` 어노테이션을 사용해도 된다.
- `ProceedingJoinPoint`의 `proceed()`는 다음 어드바이스나 타겟을 호출하는 것으로, 어드바이스를 사용하기 위해서는 꼭 해당 메소드를 호출해야 한다.
- `@Around()`라는 어노테이션이 필요하다. 이 공통 관심 사항을 어디에 적용할지 타겟팅을 해줄 수 있다.
    - 위의 경우, `hello.hellospirng`의 하위 클래스에 모두 적용한다는 것을 의미한다. 추가로, 따로 클래스 등으로 지정할 수도 있다.

---

### 스프링 빈 순환 참조

- `@Component` 어노테이션을 사용했을 때는 제대로 실행이 되었으나, 직접 `SpringConfig` 파일에 `@Bean` 어노테이션을 이용하여 등록했을 때에는 **순환 참조 문제**가 발생했다.
- 어플리케이션 컨텍스트에서 일부 `Bean`의 종속성이 순환 주기를 형성하는 문제가 발생한 것이다. 순환 종속성 또는 순환 참조 문제는 둘 이상의 `Bean`이 생성자를 통해 서로를 주입하려고 할 때 발생한다.
- 이러한 오류가 발생하는 이유는 `@Around` 코드를 보면 쉽게 이해할 수 있다. **hello.hellospring** 하위의 모든 클래스에 적용하기 때문에 `SpringConfig`의 `timeTraceAop()` 메소드에도 AOP로 처리하게 된다. 즉, 순환 참조 문제가 발생하게 된다.
- 컴포넌트 스캔을 사용할 때는 AOP의 대상이 되는 이런 코드 자체가 존재하지 않기 때문에 순환 참조 문제가 발생하지 않는 것이다.
- `SpringConfig`에 스프링 빈을 등록했을 때에도 이러한 문제가 발생하지 않도록 하려면 `@Around` 어노테이션을 사용할 때 AOP 대상에서 `SpringConfig`는 제외시키면 된다.

```java
@Around("execution(* hello.hellospring..*(..)) && !target(hello.hellospring.SpringConfig)")
```

- 위와 같이 작성해주면 `SpringConfig`는 AOP 대상에서 제외되고 순환 참조 문제는 발생하지 않게 된다.