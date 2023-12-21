### @EnableWebMvc

```java
@Target(value={TYPE})
@Import(value={DelegatingWebMvcConfiguration.class})
@Retention(value=RUNTIME)
@Documented
public @interface EnableWebMvc {

}
```

- `@EnableWebMvc` 어노테이션을 `@Configuration` 어노테이션이 사용된 클래스에 사용하면, `WebMvcConfigurationSupport`으로부터 the Spring MVC Configuration을 가져오게 된다.
- 즉, Spring에서 자동으로 여러 Config에 대한 설정을 해준다. 

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackageClasses = MyConfiguration.class)
public class MyConfiguration {

}
```
- 가져온 Configuration을 사용자 정의를 하려면 ` WebMvcConfigurer` 인터페이스를 구현하고 각각의 메소드들을 오버라이딩한다.
- 

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackageClasses = MyConfiguration.class)
public class MyConfiguration implements WebMvcConfigurer {

	   @Override
       public void addFormatters(FormatterRegistry formatterRegistry) {
         formatterRegistry.addConverter(new MyConverter());
 	   }

 	   @Override
 	   public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
         converters.add(new MyHttpMessageConverter());
 	   }

}
```

- 하나의 `@Configuration` 클래스에서만 `@EnableWebMvc`를 사용할 수 있다.
- 그러나 사용자 정의하기 위해 `WebMvcConfigurer` 인터페이스를 구현한 `@Configuration` 클래스는 여러 개가 가능하다.
- `WebMvcConfigurer`가 고급 설정을 제공하지 않는 경우에는 `@EnableWebMvc` 어노테이션을 제거하고 `WebMvcConfigurationSupport`나`DelegatingWebMvcConfiguration`에서 직접 확장할 수 있다.