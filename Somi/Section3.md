# 3. 독립 실행형 서블릿 애플리케이션

- Containerless : 서블릿 컨테이너와 관련된 번거롭고 복잡한 작업들을 개발자가 신경쓰지않고, 스프링 컨테이너에 올라가는 컴포넌트/빈을 만드는 것에만 집중해서 애플리케이션을 개발하면되도록 스프링부트가 작업해줌.

→ 메인 메서드를 실행하는 간단한 방법으로 스프링 application을 동작함.

내장형 톰캣 라이브러리 (embedded tomcat)

```java
public class Application {
		public static void main(String[] args) {
				ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
				webServer webServer = serverFactory.getWebServer();
				webServer.start();
		}
}
```

위 코드를 통해 embedded 톰캣을 띄울 수 있다.

### 서블릿 등록

서블릿 컨테이너 안에 들어가는 web component를 서블릿이라고 부른다.

- 서블릿 컨테이너가 웹 클라이언트로부터 요청을 받으면 여러개의 서블릿 중에서 어떤 서블릿에게 일을 맡길지 결정
- 이를 Mapping이라 함.

Request

- Request Line : Method, Path, HTTP Version
- Headers
- Message Body

Response

- Status Line: HTTP Version, Status Code, Status Text
- Headers
- Message Body

- ServletContextInitializr
    - Spring의 웹 모듈안에 들어있는 인터페이스
    - 서블릿 컨테이너에 서블릿을 등록하는데 필요한 작업을 수행하는 오브젝트를 만들 때 필요
    
    ```java
    public class JpashopApplication {
    
    	public static void main(String[] args) {
    		ServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
    		WebServer webServer = serverFactory.getWebServer(servletContext -> {
    			servletContext.addServlet("hello", new HttpServlet() {
    				@Override
    				protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    					resp.setStatus(200);
    					resp.setHeader("Content-Type", "text/plain");
    					resp.getWriter().println("Hello Servlet");
    				}
    			}).addMapping("/hello");
    		});
    		webServer.start();
    	}
    
    }
    ```
    
    - /hello 로 시작하는 url pattern에 매핑되는 내용들
    

### 프론트 컨트롤러

원래는 서블릿을 각 url에 매핑하여 처리하도록했었음

→ 모든 서블릿에 공통적으로 등장하는 코드를 중앙화된 제일 앞단에 존재하는 controller 오브젝트에서 공통적으로 처리하고, 요청의 종류에 따라 로직을 처리하는 다른 오브젝트한테 요청을 위임해서 전달하는 방식.
