# 4. 독립 실행형 스프링 애플리케이션

지금까지 만들었던것은 다음과 같다.

![image](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/57824857/e0a941d3-aa8e-4d79-a8eb-116df40f1fac)


서블릿 컨테이너를 띄우고, 모든 요청들을 받아서 뒷단의 오브젝트한테 적절히 위임해서 작업하도록 하는 Front Controller가 존재.

- 이제 이 hello controller라는 오브젝트를 스프링 컨테이너에 넣어보자.

## Spring Container

크게 두 가지가 필요하다.

![image](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/57824857/8dc975b5-0eef-4f98-803b-43c43cebeb2e)
- 비즈니스 로직을 담고있는 비즈니스 오브젝트, POJO.
- 만들어진 코드들을 어떤식으로 구성할지에 대한 정보를 담고있는 Configuration Metadata.
    - 어플리케이션을 어떻게 구성할것인가

### ApplicationContext

- application을 구성하고있는 정보들을 담고있는 오브젝트
- 스프링컨테이너의 대표적인 인터페이스
- GenericApplicationContext가 대표적
    - applicationContext.registerBean(HelloController.class) 와 같이 Bean을 등록한다.
    - applicationContext.refresh() : 구성정보 초기화

- Spring Container는 어떤 타입의 오브젝트를 만들 때 딱 한번만 만든다.
    - 여러개의 서블릿이 getBean을 통해 호출할 때마다 새로 오브젝트를 생성하는 것이 아니라, 같은 오브젝트를 재사용하로독 한다.
    
    ⇒ **싱글톤 패턴**과 유사.
    
- 그래서 스프링 컨테이너를 Singleton Registry라고도 함.
    - 싱글톤 패턴을 사용하지 않고도 마치 싱글톤 패턴을 쓰는것처럼 매 요청마다 새로운 오브젝트를 생성하지않고 재사용하도록 함.
- 역할에 따라서 오브젝트를 분리해서 만들고, 하나의 오브젝트가 기능을 필요로 하면 다른 오브젝트한테 요청함.

컨트롤러가 수행하는 중요한 일 중 하나는 유저의 요청사항을 검증하는 것.

예시

```java
public class HelloController {
		public String hello(String name) {
				SimpleHelloService helloService = new SimpleHelloService();
				
				return helloService.sayHello(Objects.requireNonNull(name));
		}
}
```

```java
public class SimpleHelloService {
		String sayHello(String name) {
				return "Hello " + name;
		}
}
```

## Dependency Injection

DI

![image](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/57824857/dc19fb20-3dce-429e-a5a0-60fa9a61755b)


- 위의 예시에서는 다음과 같은 의존성 관계를 가지고 있다. 즉, SimpleHelloService가 변경되면 HelloController가 영향을 받는다.
    - 이런 상황에서 **HelloController가 SimpleHelloService에 의존하고 있다**고 말할 수 있다.

![image](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/57824857/2f7f243e-379e-4eaa-8196-de0ce27a54e9)


- 이런식으로 HelloController 가 HelloService Interface에만 의존하도록 만들면, 이 서비스를 구현하는 클래스가 아무리 많아지더라도 HelloController는 변경이 없다.
- 이렇게 되면 소스코드 레벨에서는 HelloController가 특정한 클래스에 의존하지 않게 된다.
- 그러나 런타임시에는 결국 HelloService 인터페이스를 구현한 어떤 서비스의 오브젝트를 이용하도록 결정해야한다.
    - 어느 클래스의 오브젝트로 만든 메서드를 호출해야할지 알수가 없음. 따라서 둘 사이의 연관관계를 만들어줘야함.
    - 이 작업을 dependency injection이라고 하고, 이 때 Assembler를 사용한다.


![image](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/57824857/4ae05553-8cba-4641-b254-986abf007815)

- HelloController는 HelloService 를 구현한 어떤 클래스에 의존을 하는데, 소스코드레벨에서는 의존하지않고자 함. 그렇다면 런타임 시 어떤 오브젝트를 사용할지 어떻게 결정할까?
- HelloController가 사용한 오브젝트를 직접 new 키워드를 사용해서 만드는 대신, 외부에서 그 오브젝트를 만들어서 HelloController가 사용할 수 있도록 주입해줌.
    - 이 작업을 Assembler가 수행한다.

Assembler : 원래는 의존관계가 없는 클래스의 오브젝트를 가져다가 서로 관계를 연결시켜주고 사용할 수 있도록 만들어줌.

이 Assembler를 Spring Container라고 함.

- 우리가 준 Meta data를 가져다가 클래스의 singleton object를 만들고, 그 오브젝트를 사용할 다른 의존 오브젝트가 있다면 그 오브젝트를 주입해주는 과정까지 수행한다.
- 심플 헬로서비스 오브젝트 레퍼런스를 넘겨줌.

- 어떻게 넘겨줄까?
    - HelloController를 만들 때 생성자 parameter로 simplehelloService의 오브젝트를 넘겨줌. (타입은 HelloService 인터페이스) ⇒ 생성자 주입 방식
    - FactoryMethod로 빈을 만들어서 파라미터로 넘김
    - HelloController 클래스에 property 정의해서 setter로 사용할 클래스 주입
    
     생성자 주입방식 DI 예제
    
    ```java
    public class HelloController {
    		private final HelloService helloService;
    
    		public HelloController(HelloService helloService) {
    				this.helloService = helloService;
    		}
    
    		public String hello(String name) { ... }
    }
    ```
    
    - Bean 등록시에는 정확히 어떤 클래스를 가지고 bean을 만들것인지 명시
    
    <aside>
    💡 DI는 인터페이스를 통해 코드레벨의 의존관계를 제거하고, 동적으로 어셈블러가 연관관계를 주입이라는 방법을 사용해 지정하는 것이다.
    
    </aside>
    

## DispatcherServlet으로 전환

이제 ServletContainerless하게 바꿔보자

```java
public static void main(String[] args) {
		GenericWebApplicationContext applicationContext = new GenericWebApplicationContext();
		applicationContext.registerBean(HelloController.class);
		applicationContext.registerBean(SimpleHelloService.class);
		applicationContext.refresh();

		ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
		WebServer webServer = serverFactory.getWebServer(servletContext -> {
			servletContext.addServlet("dispatcherServlet", new DispatcherServlet(applicationContext).addMapping("/*");
		});
		webServer.start();
	}
```

서블릿컨테이너코드 내에 매핑정보

컨트롤러 클래스에 매핑정보를 입력해보자

```java
@GetMapping("/hello")
public String hello (String name) {
		return helloService.sayHello(Objects.requireNonNull(name));
}
```

- HTTP 메소드가 GET으로 들어오는 것 중 URL이 ‘hello’로 시작하는것을 해당 컨트롤러가 처리하겠다는 어노테이션
- dispatcherServlet은 applicationContext를 생성자로 받음 → Bean을 다 뒤져서 web요청을 처리할 수 있는 mapping정보를 가지고 있는 클래스를 찾는다.
    - 웹 컨트롤러로 판단 후, 그 안의 요청 정보를 추출한다.
    - mapping에 사용할 매핑 테이블을 생성 후, 웹 요청이 들어오면 그걸 참고해서 담당할 빈 오브젝트와 메소드를 확인함.
- 빈이 너무 많기 때문에 클래스 레벨에도 @RequestMapping어노테이션 추가해서 정보 참고. 그 후 메소드 레벨 참고

dispatcherServlet은 기본적으로 String을 return하면 리턴받은 문자열 이름의 view를 찾음

ex) “helloSpring” 을 리턴받으면 helloSpring.jsp 등 해당 이름의 view가 있는지를 체크함.

→ 따라서 뷰가 아닌 리스폰스 바디에 넣어서 응답하고싶으면 메소드위에 `@ResponseBody` 어노테이션을 추가한다.

- 또는 컨트롤러 클래스 상단에 `@RestController` 어노테이션을 붙이면 자동으로 distpatcherServlet이 모든 메소드는 지정을 하지 않는 한 `@ResponseBody`가 붙어있다고 가정한다.

## 스프링 컨테이너로 통합

서블릿 컨테이너를 만들고 서블릿을 초기화하는 작업을, 스프링 컨테이너가 초기화되는 과정중에 일어나도록 만들어보자. (스프링부트가 하는 방식)

```java
public static void main(String[] args) {
	GenericWebApplicationContext applicationContext = new GenericWebApplicationContext() {
			@Override
			protected void onRefresh() {
				super.onRefresh();

				ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
				WebServer webServer = serverFactory.getWebServer(servletContext -> {
					servletContext.addServlet("dispatcherServlet", new DispatcherServlet(this))
							.addMapping("/*");
				});
				webServer.start();
			}
		};
		applicationContext.registerBean(HelloController.class);
		applicationContext.registerBean(SimpleHelloService.class);
		applicationContext.refresh();
	}
```

- onRefresh() 하면서 초기화
- 익명클래스 사용

### Factory Method

- 어떤 오브젝트를 생성하는 로직을 담고있는 메서드
- 매핑 오브젝트를 다 생성하고 의존관계 주입. 리턴하는 오브젝트를 스프링 컨테이너에 빈으로 등록해서 사용하라고 알려줌.
- 복잡한 설정정보로 나열하는 대신, 자바 코드로 만들면 훨씬 간결해지고 이해하기 쉽기 때문에 사용.

```java
@Configuration
public class HelloBootApplication{
	
	@Bean
	public HelloController helloController(HelloService helloService) {
		return new HelloController(helloService);		
	}
	
	@Bean
	public HelloService helloService() {
		return new SimpleHelloService();
	}

	public static void main(String[] args) {
		 AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext() {
			@Override
			protected void onRefresh() {
				super.onRefresh();

				ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
				WebServer webServer = serverFactory.getWebServer(servletContext -> {
					servletContext.addServlet("dispatcherServlet", new DispatcherServlet(this))
							.addMapping("/*");
				});
				webServer.start();
			}
		};
		applicationContext.refresh();
	}
}
```

- 스프링컨테이너가 이 빈 오브젝트를 가진 클래스라는 것을 인식하도록 클래스 레벨에 `@Configuration` 어노테이션을 붙여줘야한다.
    - 스프링 컨테이너가 이 클래스 안에 빈 어노테이션이 붙은 factory 메소드가 있다는 것을 인식함.

- 이제 `AnnotationConfigWebApplicationContext` 으로 변경
    - 자바코드로 구성된 정보를 가지고 있는 클래스 이름을 등록해야함.
    - 빈은 등록X
    
- `@Configuartion`이 붙은 클래스가 AnnotationWebConfig를 이용하는 applicationContext에 처음 등록된다.
    - 전체 어플리케이션을 구성하는데 필요한 중요한 정보들을 많이 넣을 수 있기 때문.

### @Component

Spring 컨테이너에 있는 ComponentScanner를 이용해 `@Component` 어노테이션이 붙은 모든 클래스를 찾아서 빈으로 등록하게 한다.

이렇게하면 위의 방식대로 빈 등록을 직접 할 필요가 없다.

`@Configuartion` 이 붙고, Application 을 등록하는 클래스

- 여러가지 정보의 컨테이너를 구성하는데 필요한 힌트들을 넣을 수 있다.
- 여기에 `@ComponentScan` 어노테이션을 붙여 이 클래스가 있는 패키지부터 하위패키지를 스캔해서 `@Component` 어노테이션이 붙은 클래스를 빈으로 등록.
- 새로운 빈을 만들어서 추가할 때 매번 구성정보를 등록할 필요 없이, 작성하는 클래스가 빈으로 등록되어 사용될거라면 어노테이션만 붙이면 됨.
- 만약 빈으로 등록되는 클래스가 많아지면 나중에 정확히 어떤 클래스들이 등록되었는지 찾아보기가 어려울 수 있다.

@Component를 직접 클래스에 붙여도 되지만, 이 어노테이션을 `메타 어노테이션`으로 가지고 있는 어노테이션을 붙여도 `@Component`가 붙은것과 동일한 효과를 가짐.

- Meta Annotation : 어노테이션 코드 위에 또 어노테이션을 붙인 것.
- ex) `@Service`, `@Controller`, `@Repository`

어노테이션을 만들어보자

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Component
public @interface MyComponent {
}
```

- `@Retention` : 이 어노테이션이 어디까지 살아있을것인가. (역시 메타 어노테이션)
- `@Target` : 어노테이션을 적용할 대상을 지정

위의 작업들을 run 메서드로 따로 빼서 실행하는 것이 스프링 어플리케이션 실행 코드

```java
public class BatchApplication {
	public static void main(String[] args) {
		SpringApplication.run(BatchApplication.class, args);
	}
}
```

- 서블릿컨테이너가 뜨고 스프링컨테이너 만들어지고 동작하는 과정들.
- 스프링 부트가 stand alone으로 서블릿컨테이너까지 포함하는 스프링 어플리케이션을 동작시키는 원리가 담겨져있는 코드.
