# 섹션 6 - 스프링 부트 외관 완성

# STEP 0. 현재 코드

지금까지 만든 우리 코드를 보면 꽤 비슷해졌지만 스프링 부트 어플리케이션에 비하면 부수적인게 아직 많이 있는 걸 볼 수 있다. 이번 장에서는 이를 거의 비슷한 모습으로 완성 시킬 것이다.

- 우리 코드

```java
@Configuration
@ComponentScan
public class HellobootApplication {

    @Bean
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    public static void main(String[] args) {
        SpringApplication.run(HellobootApplication.class, args);
    }
}
```

- 스프링 부트 어플리케이션

```java
@SpringBootApplication
public class HellobootApplication {

  public static void main(String[] args) {
    SpringApplication.run(HellobootApplication.class, args);
  }
}
```

# 개념 - 메타 어노테이션

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/2d987b8f-27cf-4a4b-9ad5-f031dc5a1537)

`@Controller`, `@Service` 는 메타 어노테이션으로 `@Component` 를 가지고 있다.

`@Service` 를 빈으로 등록해주기 위해 `@Component`를 가지고 있는 것.

## 예제

- `@Test` 를 메타로 가지고 있는 `@UnitTest`
- `@UnitTest`를 메타로 가지고 있는 `@FastUnitTest`

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@UnitTest
@interface FastUnitTest {
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD})
@Test
@interface UnitTest {
}
```

# STEP 1. 메타 어노테이션 활용

`@Configuration`, `@ComponentScan` 에 담긴 어노테이션을 묶은 `@MySpringBootApplication` 을 만들어 붙혀주자.

```java
@Retention(RetentionPolicy.RUNTIME) // 런타임까지 어노테이션 정보가 유지되도록
@Target(ElementType.TYPE) // class, interface enum에 적용될 어노테이션
@Configuration  // 메타 어노테이션
@ComponentScan  // 메타 어노테이션
public @interface MySpringBootApplication {
}

@MySpringBootApplication
public class HellobootApplication {

  @Bean
  public ServletWebServerFactory servletWebServerFactory() {
    return new TomcatServletWebServerFactory();
  }

  @Bean
  public DispatcherServlet dispatcherServlet() {
    return new DispatcherServlet();
  }

  public static void main(String[] args) {
    SpringApplication.run(HellobootApplication.class, args);
  }

}
```

# STEP 2. config 클래스 추가

`@Configuration` 어노테이션을 가진 config 클래스를 추가하여 servlet 관련 빈 등록 로직을 옮겨주면 스프링 부트 로직과 거의 비슷해 진 것을 볼 수 있다.

```java
@Configuration
public class Config {

	@Bean
	public ServletWebServerFactory servletWebServerFactory() {
		return new TomcatServletWebServerFactory();
	}

	@Bean
	public DispatcherServlet dispatcherServlet() {
		return new DispatcherServlet();
	}
}

@MySpringBootAnnotation
public class HellobootApplication {

	public static void main(String[] args) {
		SpringApplication.run(HellobootApplication.class, args);
	}
}
```
