> JPA는 객체-관계 매핑(ORM)을 위한 표준 명세인 인터페이스를 의미한다. 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스와 어노테이션의 표준 집합을 정의한다. 즉, 특정 기능을 위한 라이브러리가 아니라 인터페이스이다.

> Spring Data JPA는 JPA 기반 어플리케이션 개발을 보다 간편하게 만드는 라이브러리, 프레임워크이다. JPA에서는 `EntityManager`를 사용하지만 Spring Data JPA에서는 `Repository`라는 인터페이스를 제공한다.


```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
		// select m from Member m where m.name = ?
    Optional<Member> findByName(String name);
}
```

- 현재 인터페이스만 생성해 놓은 상태인데 `JpaRepository`를 상속 받고 있으면 스프링에서 자동으로 구현체를 생성하여 스프링 빈으로 등록해준다.

```java
private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository);
    }
```

- 그런 후에 `SpringConfig` 파일에 다음과 같이 작성해주면 된다. 그러면 스프링에서 `JpaRepository`를 상속받은 인터페이스의 구현체를 `memberRepository`에 넣어준다. 그리고 `MemberService`와의 의존관계도 설정해주면 된다.
- 위에서 생성자가 하나만 있는 경우에는 `@Autowired`를 생략해도 된다.
- 따라서 원래 따로 `@Bean` 어노테이션을 이용하여 스프링 빈을 등록하는 아래의 코드는 불필요해진다. (주석 처리함.)

```java
//    @Bean
//    public MemberRepository memberRepository(){
//        return new MemoryMemberRepository();
//        return new JdbcMemberRepository(dataSource);
//        return new JdbcTemplateMemberRepository(dataSource);
//        return new JpaMemberRepository(em);
```

- 그렇다면 왜 메소드는 `findByName` 하나만 만들었냐 한다면 이미 `JpaRepository`에서 CRUD와 관련된 메소드들을 제공하고 있기 때문이다. `save`, `findAll`, `findById` 등 이미 우리가 그동안 하나하나 만들어왔던 메소드들을 정의해 놓았다.
- 인터페이스에서 제공하지 않는 메소드들은 개발자가 직접 정의해서 사용할 수 있는 것이다.
- 만약 `Member` 엔티티에 `name`, `email` 과 같은 속성이 있다면 `findByName`, `findByEmail`과 같이 메소드의 이름을 작성하면 스프링에서 메소드의 이름만을 보고 `JPQL` 쿼리를 작성해준다. 그리고 그 쿼리를 `SQL`로 변환해준다.