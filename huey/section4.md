
# STEP 1. 스프링 컨테이너 추가

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/f3f4174a-2952-4a5b-b110-d1908589e90a)

프론트 컨트롤러가 HelloController를 직접 생성하고 변수에 담아 실행하는 대신 스프링 컨테이너 안에 HelloController를 넣고 이용하는 방식으로 변경할 것이다.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/c37fd042-74be-4854-8d39-4ec75e2da7b4)

## 스프링 컨테이너를 사용했을 때의 장점 (feat. 싱글톤 패턴)

- 결과적으로 보면 FrontController가 직접 HelloController를 생성해서 사용하는 것과 다를게 없다.
- 스프링 컨테이너는 기본적으로 어떤 타입의 오브젝트를 만들 때 딱 한번만 만든다.
- 추후 여러 서블릿이 생긴다면 HelloController가 여러번 생성되는 경우가 생긴다.
- 따라서 여러 서블릿이 하나의 HelloController를 사용할 수 있도록 Spring Container에 만들어 등록해 놓는 것. → **싱글톤 패턴**
    - 이런 특징 때문에 스프링 컨테이너를 **싱글톤 레지스트리** 라고도 부른다.

# STEP 2. Service 추가

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/7f5cae2d-9ddf-441e-a2dd-9dd12ad2eeef)

HelloController는 웹 컨트롤러로써 요청 검증 후 비즈니스 로직을 제공해주는 오브젝트에게 요청을 전달하여 응답값을 적절하게 반환하는 역할만 수행하게끔 만들자.

그리고 SimpleHelloService 빈을 하나 더 등록하여 비즈니스 로직을 넣어주자.

```java
public class HellobootApplication {
	public static void main(String[] args) {
	  // 스프링 컨테이너
	  GenericApplicationContext applicationContext = new GenericApplicationContext();
	  // 빈 등록
	  applicationContext.registerBean(HelloController.class);
	  applicationContext.registerBean(SimpleHelloService.class);
	  // 컨테이너 초기화
	  applicationContext.refresh();
		...
	}
}

public class HelloController {
  private final HelloService helloService;

	// 생성자를 통해 의존성 주입
  public HelloController(HelloService helloService) {
    this.helloService = helloService;
  }

  public String hello(String name) {
		// null 체크 후 메소드 요청
    return helloService.sayHello(Objects.requireNonNull(name));
  }
}

public interface HelloService {
  String sayHello(String name);
}

public class SimpleHelloService implements HelloService {
  @Override
  public String sayHello(String name) {
		// 비즈니스 로직 수행
    return "Hello " + name;
  }
}
```

# STEP 3. DispatcherServlet 전환

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/a4f9f5bd-839c-44f7-9272-992078484b30)

- DispatcherServlet은 이전에 만든 FrontController의 작업을 수행해준다. (매핑, 바인딩 등)
- 당연히 스프링 컨테이너를 알고 있어야 하기 때문에 생성자에 스프링 컨테이너를 넣어준다.
- 위 코드를 실행하고 `/hello?name=toby` url에 요청을 보내면 404 코드가 뜨는데 바인딩과 매핑을 위한 정보가 아직 없기 때문이다.

## 어노테이션 매핑 정보 사용

- DispatcherServlet에 매핑 정보와 바인딩 정보를 넣어주기 위해서 예전에는 xml에 일일이 작성하였다.
- 요즘에는 보통 **controller에 매핑 정보를 명시하는 어노테이션 매핑** 방법을 사용한다.

```java
@RequestMapping("/hello") // 매핑 정보 추가
public class HelloController {
  private final HelloService helloService;

	// 생성자를 통해 의존성 주입
  public HelloController(HelloService helloService) {
    this.helloService = helloService;
  }

	@GetMapping // 매핑 정보 추가
	// @RequestMapping(value = "/hello", method = RequestMethod.GET)
	@ResponseBody // body 응답
  public String hello(String name) {
		// null 체크 후 메소드 요청
    return helloService.sayHello(Objects.requireNonNull(name));
  }
}
```

DispatcherServlet은 요청에 대한 올바른 메소드를 선택하게 되는데 클래스를 먼저 탐색하고 메소드를 탐색하기 때문에 클래스 레벨에도 `@RequestMapping` 어노테이션을 붙혀주자.

- `@RestController` 어노테이션을 붙히면 `ResponseBody`를 생략해도 된다.

# STEP 4. 스프링 컨테이너로 통합

## Before

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/422f1ecf-7481-4685-a9e3-2c8083f4c7ad)

현재 코드는 크게 두개의 과정으로 이뤄져있다. 이걸 하나의 과정으로 통합하여 서블릿 컨테이너 초기화를 스프링 컨테이너 초기화 과정 중에 진행하도록 수정하자. (이유는 넘 복잡해서 skip)

## After

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/74f87f7c-3281-4598-862f-0fb52a01092b)

# STEP 5. `@Configuration` 구성정보 사용

빈 사이의 의존관계, 언제 주입해야 할지 등을 스프링 컨테이너에 구성정보로 제공해야 한다.

따라서 구성정보가 담겨있는 클래스를 선언하고 스프링 컨테이너에게 알리자.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/76177d46-54ef-417c-8552-41ea7b976b2a)

# STEP 6. `@Component` 구성정보 사용 (`@Controller`, `@Service`)

- STEP 5 보다 간단한 방법은 빈으로 등록될 클래스에 직접 빈임을 표시하는 것이다. `@Component` 어노테이션을 빈으로 등록할 클래스에 붙혀주면, 스프링 컨테이너가 컴포넌트 스캔 작업을 통해 빈으로 등록된다.
- 간단하다는 장점이 있지만, 나중에 빈이 많아지면 앱이 실행됐을 때 어떤 빈이 등록되는지 찾기 어려워진다. 하지만 이 단점도 패키지를 잘 나누면 해결할 수 있는 문제이기 때문에 해당 방식이 표준으로 여겨진다.
- `@RestController`, `@Controller` , `@Service` 는 `@Component` 어노테이션이 포함된 어노테이션이다.
    - `@RestController` = `@Controller` + `@ResponseBody`
    - 또한 `@RestController`, `@Controller` 를 붙혀주면 DispatcherServlet이 매핑 작업을 할 때 해당 어노테이션이 붙어 있으면 그 안에 있는 메소드를 우선 탐색하기 때문에 STEP 3 에서 언급했던 ‘클래스 레벨에도 RequestMapping을 붙혀주는게 좋다’가 생략이 가능하다.

```java
@Configuration
@ComponentScan  // 컴포넌트 스캔 요청
public class HellobootApplication {...}

@RestController 
public class HelloController {...}

@Service
public class SimpleHelloService implements HelloService {...}
```

# STEP 7. servlet도 빈으로 등록 (Bean 생명주기)

현재 메인 메소드에 있는 Servlet 관련 오브젝트도 빈으로 등록하고자 한다. 장점은 추후 강의에서…

`TomcatServletWebServerFactory`, `DispatcherServlet` 을 STEP 5 방식을 통해 빈으로 등록해보자.

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/04e1ce91-92a1-4038-84c8-61b79ecea7a7)

주석 부분을 보면 DispatcherServlet에 ApplicationContext(스프링 컨테이너)를 넣어주는 부분이 없어도 정상 동작을 한다. → 스프링 컨테이너가 DispatcherServlet이 대신 주입하기 때문.

# STEP 8. 재사용 가능한 클래스로 분리

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/2ae3aa82-1529-4ee6-8ef0-9c6956fc4c8c)

main 메소드에 있던 작업을 새로 생성한 `MySpringApplication`클래스의 run 메소드로 만들고, 구성정보가 담긴 클래스와 별도의 args를 받도록 하자.

그러면 아래와 같이 우리가 알던 스프링부트와 굉장히 비슷해진다!

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/b74acbfc-49a8-4510-bf18-1f5573623298)