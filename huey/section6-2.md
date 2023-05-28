# 섹션 6 구성정보

# 빈(Bean) 오브젝트의 역할과 구분

빈은 아래와 같이 나눌 수 있다.

- 애플리케이션 로직 빈
    - 개발자가 명시적으로 구성정보로 제공한 빈 오브젝트
    - 실제 로직이 담겨있음
- 애플리케이션 빈
    - 개발자가 명시적으로 구성정보로 제공한 빈 오브젝트
    - 기술적으로 필요한 것들
- 컨테이너 인프라스트럭처 빈
    - 스프링 컨테이너 자신, 또는 컨테이너 스스로 빈으로 등록해서 사용하는 빈 오브젝트

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/db3d19e1-22b7-4951-8b01-82a5ee7c1dd2)

그리고 구성정보로써의 빈으로는 아래 두 종류로 나눌 수 있다.

- 사용자 구성정보
    - 어플리케이션의 로직을 담당
    - 주로 컴포넌트 스캔에 의해 빈으로 등록됨
- 자동 구성정보
    - 기술과 관련된 빈들
    - 스프링 부트가 자동으로 등록시킴

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/77793e0c-1818-46c2-8407-57b621827fa0)

# STEP 1. 구성정보 분리

지금의 코드를 보면 모든 빈 들이 컴포넌트 스캔의 대상이다. (`@Configuration` 은 컴포넌트 스캔의 대상이기 때문)

이를 컴포넌트 스캔 대상에서 제외시키고 자동 구성정보로 등록하자.

1. 컴포넌트 스캔의 대상이 되는 루트 패키지 밖으로 제외하기 위해 `HelloBootApplication`이 속한 패키지 밖에 config 패키지를 만들고 Config 클래스를 옮기자
2. 하나의 config에 두 개의 빈이 등록되므로 파일을 분리하고 config/autoconfig 로 이동 및 분리
3. `@EnableMyAutoConfiguration`어노테이션을 추가하고, `@Import` 를 통해 하드코딩 형식으로 Config 클래스를 구성정보로 가져오자.
4. 그리고 이를 `@MySpringBootApplication` 어노테이션에 메타 어노테이션으로 붙혀주자.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/ef86bff8-34c0-49bc-b282-168f22de2a0b)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import({DispatcherServletConfig.class, TomcatWebServerConfig.class})
public @interface EnableMyAutoConfiguration {
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration
@ComponentScan
@EnableMyAutoConfiguration
public @interface MySpringBootApplication {
}
```

아래 그림과 같이 컴포넌트 스캔 대상에서 분리시켰고, `@EnableMyAutoConfiguration` 와 `@Import`를 통해 구성정보로 등록되는 것을 볼 수 있다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/a167faeb-1dc5-4b17-912b-0213ee81422e)

# STEP 2. 동적인 자동 구성정보 - ImportSelector, 구성정보 분리

현재는 `@Import` 로 하드하게 직접 가져오는데 이를 동적으로 가져올 수 있도록 수정하자.

- 기존에 있던 `@EnableMyAutoConfiguration` 가 직접 `DispatcherServletConfig`, `@TomcatWebServerConfig` 를 import 하는게 아니라 중간에 `MyAutoConfigImportSelector` 라는 Selector를 두어 동적으로 import 할 수 있도록 수정한다.
- import 할 클래스의 경로가 담긴 imports 파일을 추가하고 해당 imports 파일과 같은 이름의 인터페이스를 생성한다.
    - 클래스 추가
    
    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @Configuration(proxyBeanMethods = false)
    public @interface MyAutoConfiguration {
    }
    ```
    
    - resources/META-INF.spring/tobyspring.config.MyAutoConfiguration.imports 추가
    
    ```
    tobyspring.config.autoconfig.TomcatWebServerConfig
    tobyspring.config.autoconfig.DispatcherServletConfig
    ```
    
- Selector가 imports 파일에 저장한 클래스들을 String으로 받아와 반환하도록 수정한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyAutoConfigImportSelector.class)  // 직접 import가 아닌 selector를 가져옴
public @interface EnableMyAutoConfiguration {
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Configuration(proxyBeanMethods = false)
public @interface MyAutoConfiguration {
}

public class MyAutoConfigImportSelector implements DeferredImportSelector {

  private final ClassLoader classLoader;

  public MyAutoConfigImportSelector(ClassLoader classLoader) {
    this.classLoader = classLoader;
  }

  @Override
  public String[] selectImports(AnnotationMetadata importingClassMetadata) {
    List<String> autoConfigs = new ArrayList<>();

    ImportCandidates.load(MyAutoConfiguration.class, classLoader).forEach(autoConfigs::add);

    return autoConfigs.toArray(new String[0]);
  }
}
```

그림으로 보면 중간에 Selector가 추가되었고 Import 할 Config파일 정보가 외부 설정파일에 존재하게 된 것을 볼 수 있다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/21bcf948-9eaf-450a-a569-662c8fcc22b8)

# +a. @Configuration(proxyBeanMethods = false)

```java
public class ConfigurationTest {

  @Test
  void 일반호출() {
    MyConfig myConfig = new MyConfig();

    Assertions.assertThat(bean1.common).isSameAs(bean2.common);
  }     // 테스트 실패

  @Test
  void 빈호출() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
    ac.register(MyConfig.class);  // 컨텍스트에 빈으로 등록
    ac.refresh();

    Assertions.assertThat(bean1.common).isSameAs(bean2.common);
  }     // 테스트 성공

  @Configuration
  static class MyConfig {

    @Bean
    Common common() {
      return new Common();
    }
    @Bean
    Bean1 bean1() {
      return new Bean1(common());
    }
    @Bean
    Bean2 bean2() {
      return new Bean2(common());
    }
  }

  static class Bean1 {
    private final Common common;

    Bean1(Common common) {
      this.common = common;
    }
  }

  static class Bean2 {
    private final Common common;

    Bean2(Common common) {
      this.common = common;
    }
  }

  static class Common {
  }

}
```

일반 호출로 했을 때에는 당연히 new가 두번 호출 되어 bean1과 bean2가 다르지만, 빈으로 등록하여 비교하면 같다고 나온다.

컨텍스트에 등록했을 때 내부에서 아래와 같은 로직으로 등록된다. (프록시 패턴)

```java
static class MyConfigProxy extends MyConfig {

  private Common common;

  @Override
  Common common() {
    if (this.common == null) {
      this.common = super.common();
    }

    return this.common;
  }
}
```

**→ 스프링은 `@Configuration` 어노테이션이 붙은 클래스는 기본적으로 프록시를 만들어서 기능을 확장해 줌.**

이러한 기능을 끄는 기능은 Spring 5.0에 추가된 옵션값을 활용하면 된다.

`@Configuration(proxyBeanMethods = false)`
