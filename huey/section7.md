# 섹션 7 - 구성정보

현재 코드를 보면 `@MyAutoConfiguration` 어노테이션이 `MyAutoConfiguration.imports`에 저장된 구성 정보를 담고 있고, `MyAutoConfigImportSelector` 클래스가 이를 받아온 후, `@EnableMyAutoConfiguration`으로 자동 구성 정보가 등록된다.

실제 스프링 부트 코드에는 똑같이 `@AutoConfiguration`, `AutoConfiguration.imports` 이 존재하는데, 사용하지 않는 구성정보가 전부 포함되어 있는 걸 볼 수 있다. (100개 이상의 구성정보가 등록되어 있음)

이를 스프링 부트가 동적으로 필요한 구성정보만 빈으로 등록하는데, 현재 코드에 톰캣 외의 다른 종류의 서블릿 컨테이너인 Jetty를 구성정보로 추가하여 동적 빈 구성이 되도록 수정해보자.

# 예제 STEP 1. Jetty 추가

- build.gradle 의존성 추가

```java
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-jetty'  // 추가
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

- config/autoconfig/JettyWebServerConfig.class 추가

```java
@MyAutoConfiguration
public class JettyWebServerConfig {

  @Bean("jettyWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new JettyServletWebServerFactory();
  }
}

@MyAutoConfiguration
public class TomcatWebServerConfig {

  @Bean("tomcatWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new TomcatServletWebServerFactory();
  }
}
```

- MyAutoConfiguration.imports 추가

```
tobyspring.config.autoconfig.TomcatWebServerConfig
tobyspring.config.autoconfig.JettyWebServerConfig     <- 추가
tobyspring.config.autoconfig.DispatcherServletConfig
```

이렇게 Jetty를 구성정보에 추가하고 서버를 실행하면 서블릿 컨테이너가 두 종류(tomcat, jetty)이기 때문에 에러가 발생한다.

`Unable to start ServletWebServerApplicationContext due to multiple ServletWebServerFactory beans : tomcatWebServerFactory, jettyWebServerFactory`

# 예제 STEP 2. Conditional을 통한 동적 구성정보

`Condition.matches` 의 반환값에 따라 해당 빈을 등록할지를 결정한다.

```java
@MyAutoConfiguration
@Conditional(JettyWebServerConfig.JettyCondition.class)  // 추가
public class JettyWebServerConfig {

  @Bean("jettyWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new JettyServletWebServerFactory();
  }

  static class JettyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      return false;   // false 이므로 빈 등록 X
    }
  }
}

@MyAutoConfiguration
@Conditional(TomcatWebServerConfig.TomcatCondition.class)
public class TomcatWebServerConfig {
  ..// return true;
}
```

이후 서버를 실행하면 서버가 정상 작동. Tomcat 만 빈에 등록되기 때문

이 처럼 구성정보에는 분명 둘 다 있지만, Condition 코드를 통해 빈에 동적으로 등록될 수 있다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/94e05621-1e16-498b-9fb0-f5d25c65834f)

# 예제 STEP 3. Condition 로직 적용

Tomcat 과 Jetty 중 어떤 서블릿 컨테이너를 빈으로 등록할지 결정해야 함.

dependecy에 추가되어 프로젝트에 해당 패키지(클래스)가 존재하는지 여부로 결정.

```java
@MyAutoConfiguration
@Conditional(JettyWebServerConfig.JettyCondition.class)
public class JettyWebServerConfig {

  @Bean("jettyWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new JettyServletWebServerFactory();
  }

  static class JettyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      return ClassUtils.isPresent("org.eclipse.jetty.server.Server",
                context.getClassLoader());  // 로직 추가
    }
  }
}

@MyAutoConfiguration
@Conditional(TomcatWebServerConfig.TomcatCondition.class)
public class TomcatWebServerConfig {

  @Bean("tomcatWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new TomcatServletWebServerFactory();
  }

  static class TomcatCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
      return ClassUtils.isPresent("org.apache.catalina.startup.Tomcat",
                context.getClassLoader());  // 로직 추가
    }
  }
}
```

# 예제 STEP 4. 어노테이션으로 중복 코드 제거

Condition 코드가 항상 중복될 것이므로 이를 어노테이션으로 선언해 사용하도록 하자.

```java
// 커스텀 어노테이션 추가
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Conditional(MyOnClassCondition.class)
public @interface ConditionalMyOnClass {
  String value();
}

public class MyOnClassCondition implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    Map<String, Object> attrs = metadata.getAnnotationAttributes(ConditionalMyOnClass.class.getName());
    String value = (String) attrs.get("value");
    return ClassUtils.isPresent(value, context.getClassLoader());
  }
}

@MyAutoConfiguration
@ConditionalMyOnClass("org.eclipse.jetty.server.Server")
public class JettyWebServerConfig {
  @Bean("jettyWebServerFactory")
  public ServletWebServerFactory servletWebServerFactory() {
    return new JettyServletWebServerFactory();
  }
}
```

![Untitled 1](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/3156e912-2239-4e41-a622-ff5e718ec5f1)
