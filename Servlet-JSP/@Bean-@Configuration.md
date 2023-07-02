## @Bean
- 스프링은 객체를 생성하고 초기화하는 기능을 제공
- 스프링이 생성하는 객체가 빈(Bean) 객체
- `@Bean` 어노테이션을 메소드에 붙이면 해당 메소드가 생성한 객체를 스프링이 관리하는 빈 객체로 등록
- 해당 어노테이션을 붙인 메소드의 이름은 빈 객체를 구분할 때 사용
- 스프링은 기본적으로 한 개의 `@Bean` 어노테이션에 대해 하나의 객체를 생성

---
## @Configuration
- `@Configuration` 어노테이션은 해당 클래스를 스프링 설정 클래스로 지정

---
## AnnotationConfigApplicationContext
- 자바 설정에서 정보를 읽어와 객체를 생성하고 관리
- `@Configuration` 어노테이션이 붙은 클래스를 설정 정보로 사용
- `getBean()` 메소드는 AnnotationConfigApplicationContext가 자바 설정을 읽어와 생성한 빈 객체를 검색할 때 사용
    1. 첫 번째 파라미터: `@Bean` 어노테이션의 메소드 이름인 빈 객체의 이름
    2. 두 번째 파라미터: 검색할 빈 객체의 타입