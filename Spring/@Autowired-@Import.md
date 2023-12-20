### @Autowired (자바 코드로 스프링 빈 등록)
- 스프링의 자동 주입 기능을 위한 것
- 스프링 설정 클래스의 필드에 `@Autowired` 어노테이션을 붙이면 해당 타입의 빈을 찾아서 필드에 할당

```java
@Configuration
public class AppConf1 {
	
	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberPrinter memberPrinter() {
		return new MemberPrinter();
	}

}
```

```java
@Configuration
public class AppConf2 {
	
	@Autowired
	private MemberDao memberDao;
	@Autowired
	private MemberPrinter memberPrinter;

    @Bean
	public MemberInfoPrinter infoPrinter() {
		MemberInfoPrinter infoPrinter=new MemberInfoPrinter();
		infoPrinter.setMemberDao(memberDao);
		infoPrinter.setMemberPrinter(memberPrinter);
		return infoPrinter;
	}
}
```
- MemberDao 타입의 빈을 memberDao 필드에, MemberPrinter 타입의 빈을 memberPrinter 필드에 할당

---
- 위의 MemberInfoPrinter 클래스에 `@Autowired` 어노테이션을 사용하면 스프링 설정 클래스의 `@Bean` 메소드에서 의존 주입을 위한 코드 삭제 가능

```java
@Bean
	public MemberInfoPrinter infoPrinter() {
		MemberInfoPrinter infoPrinter=new MemberInfoPrinter();
		// infoPrinter.setMemberDao(memberDao);
		// infoPrinter.setMemberPrinter(memberPrinter);
        // 위 두 줄의 코드를 작성하지 않아도 됨
		return infoPrinter;
	}
```

---
### @Import (참고)
두 개 이상의 설정 파일을 사용하는 또 다른 방법

```java
@Configuration
@Import(AppConf2.class)
public class AppConfImport {
	
	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberPrinter memberPrinter() {
		return new MemberPrinter();
	}

}
```
- AppConfImport 설정 클래스를 사용하면 `@Import` 어노테이션으로 지정한 AppConf2 설정 클래스도 사용
- 스프링 컨테이너를 생성할 때 AppConf2 설정 클래스를 지정할 필요 X

```java
ctx=new AnnotationConfigApplicationContext(AppConfImport.class);
```