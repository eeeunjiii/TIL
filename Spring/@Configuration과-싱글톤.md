### `@Configuration`과 싱글톤

```java
@Configuration
public class AppConfig {
    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```
- 만약 `memberService`와 `orderService`를 호출하면 각각 `memberRepository()`를 호출하여 `return new MemoryMemberRepository()`를 실행하게 된다.
- 즉 `MemoryMemberRepository` 객체를 두 개 생성되어 싱글톤 패턴이 깨지는 것처럼 보인다.

```java
public class ConfigurationSingletonTest {

    @Test
    void configurationTest(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberService -> memberRepository1 = " + memberRepository1);
        System.out.println("orderService -> memberRepository2 = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        Assertions.assertThat(memberRepository1).isSameAs(memberRepository);
        Assertions.assertThat(memberRepository2).isSameAs(memberRepository);
    }
}
```
- 그러나 테스트를 통해 `MemoryMemberRepository`에 대해 하나의 객체만 생성되어 모두 동일한 것임을 알 수 있다.
- `@Configuration` 어노테이션이 빈을 등록할 때 싱글톤이 되도록 보장해주는 역할을 한다.
  - 스프링 컨테이너에서 빈을 관리할 수 있게 된다.
- '싱글톤 방식의 주의점'에서 `TestConfig`에는 `@Configuration`이 붙지 않아도 동일한 객체를 반환했다.
  - 그 이유는 `StatefulService`를 한 번만 호출하여 그에 대한 빈이 하나만 만들어졌기 때문에 조회할 때에도 생성된 하나의 빈 객체를 반환한 것이다.
  - `@Configuration`을 붙여야 맞는 것이다.