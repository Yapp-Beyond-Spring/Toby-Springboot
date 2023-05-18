# 5. DI와 테스트, 디자인 패턴

매번 어플리케이션을 실행해서 테스트할 수는 없으니 테스트코드를 작성해서 테스트해보자.

```java
public class HelloApiTest {
    @Test
    void helloApi() {
        // http localhost:8080/hello?name=Spring
        TestRestTemplate rest = new TestRestTemplate();

        ResponseEntity<String> res = rest.getForEntity("http://localhost:8080/hello?name={name}", String.class, "Spring");

        assertThat(res.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(res.getHeaders().getFirst(HttpHeaders.CONTENT_TYPE)).startsWith(MediaType.TEXT_PLAIN_VALUE);
        assertThat(res.getBody()).isEqualTo("Hello Spring");
    }
}
```

api 테스트. 네트워크 구간을 통해 요청, 응답 파싱

### DI와 단위테스트

```java
@Test
void simpleHelloService() {
    SimpleHelloService helloService = new SimpleHelloService();

    String ret = helloService.sayHello("Test");

    Assertions.assertThat(ret).isEqualTo("Hello Test");
}
```

- 자바 클래스에 인스턴스 하나 생성해서 메서드 호출, 검증
- 작업이 간결하고 테스트 수행 속도가 위의 코드보다 1/10 정도

클래스를 직접 테스트하는 것이 간결하다. 또한 고립된 테스트가 가능하다는 장점이 있다.

```java
@Test
void failsHelloController() {
    HelloController helloController = new HelloController(name -> name);

    Assertions.assertThatThrownBy(() -> {
        helloController.hello(null);  // null이 넘어갔을 때 exception 발생하는지 테스트
    }).isInstanceOf(IllegalArgumentException.class);

    Assertions.assertThatThrownBy(() -> {
        helloController.hello("");  // 공백문자가 넘어갔을 때 exception 발생하는지 테스트
    }).isInstanceOf(IllegalArgumentException.class);
}
```

예외를 검증하는 테스트코드

- `IllegalArgumentException`타입의 예외가 나타나는지 검증
- 이런 테스트를 유닛테스트 (단위테스트)라고 한다.
- 이런식으로 컨테이너 없이 테스트코드를 작성하면 수행속도가 매우 빠르다.
