
```java
@SpingBootApplication
public class HelloApplication {
		public static void main(String[] args) {
				SpringApplication.run(HelloApplication.class, args);
		}
}
```

위와 같은 코드만 작성했는데 톰캣과 스프링 컨테이너가 자동으로 떠서 동작을 하고, Controller의 메소드가 알아서 매핑된다.

```java
public class HelloApplication {
		public static void main(String[] args) {
		}
}
```

이번 장에서는 Spring Boot의 도움 없이 위 코드와 똑같이 동작하도록 만들어 볼 것이다.

# STEP 1. 빈 Servlet Container 만들기

![Untitled 0](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/cd294ed7-defa-4338-a18d-03f4cbecfef2)

```java
public class HelloApplication {
		public static void main(String[] args) {
				ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
				WebServer webServer = serverFactory.getWebServer();
				webServer.start();
		}
}
```

- `ServletWebServerFactory` 는 추상화 되어있는 인터페이스.
    - `JettyServletWebServerFactory`처럼 다른 제품을 사용할 수 있음.
- `WebServer` 가 여기서는 톰캣이며, start()메소드로 서블릿 컨테이너를 실행시킴.
- 8080에 접속해보면 404 not found (톰캣이 정상적으로 실행됨)

# STEP 2. Servlet 만들기

![Untitled 1](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/23b6fdae-2e5d-4d08-89cd-61cc6f42f784)

![Untitled 2](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/e9e04f3d-1118-4c2c-aa13-822b74be10b3)

- `getWebServer()` 의 파라미터인 `ServletContextInitializer`를 통해 Servlet을 넣어준다.
    - 예제 코드에서는 람다식으로 추가함.
- hello라는 이름의 익명 클래스를 추가했으며, service를 오버라이드하여 응답을 만듦.
- addMapping을 통해 url 매핑함.
- req.getParameter()를 통해 쿼리스트링 받아옴.

# STEP 3. Front Controller 추가

기능이 추가될 때 마다 servlet을 계속 추가하면 되지만, 중복되는 부분이 많았다.

→ 프론트 컨트롤러를 추가해 중복 코드를 줄이자!

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/26a98c0c-cf56-4fe6-944e-3a37a4528286)

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/fcab8e81-30be-453e-80d6-71be7098911f)    

# STEP 4. Controller 매핑

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/92189d17-9627-4b80-a8ce-4ffe3fc7fbbf)

![Untitled](https://github.com/Yapp-Beyond-Spring/Toby-Springboot/assets/77145383/2e20f60f-8d16-4a1a-b634-d8b57be07181)

- FrontController에서 공통 로직을 수행하고, 요청을 분석해 특정 클래스의 메소드에 매핑해주고 응답한다.
- name이라는 파라미터를 String이라는 타입으로 받아서 메소드에 전달해 주는 것을 **바인딩**이라 한다.