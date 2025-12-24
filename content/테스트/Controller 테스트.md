- HTTP + Tomcat + Spring + Controller + Service 까지 많은 것이 상호작용하는 레이어라, 테스트 방식도 다채롭다.
- 각 방식의 목적이 다르므로, 테스트 목적에 맞게 진행하면 좋을 것 같다.
### 단위 테스트
- Controller만 테스트하는 방법이다.
- Controller + Mock(Service)로 진행한다.
- 의견 : MVC 쪽 기능이 아예 없으니, 사실상 검증되는게 없다.
### @WebMvcTest
- 디스패쳐 서블릿까지 띄우고, 테스트 진행시 모킹된 요청 객체를 생성하여 디스패쳐 서블릿에 직접 주입하는 방식이다.
- 슬라이스 테스트라서, Controller/ControllerAdvice 단까지만 뜬다. -> Service 레이어는 목빈이 필요하다.
- `MockMvc`를 통해서 웹 인터페이스의 역할(URL 매핑, 유효성 검증, 상태 코드) 등을 가장 가볍게 테스트할 수 있다.
```java
mockMvc.perform(post("/api/users")
			.contentType(MediaType.APPLICATION_JSON)
			.content(objectMapper.writeValueAsString(request)))
	.andExpect(status().isOk())
	.andExpect(jsonPath("$.id").value(1L))
	.andExpect(jsonPath("$.name").value("테스터"));
```
### @SpringBootTest(Mock)
- 모든 스프링을 띄우지만, 톰캣 부분은 모킹한다. (따라서 네트워크를 타지 않는다.)
- 즉 WebMvcTest(슬라이스 테스트)에 Service 레이어 이후가 같이 뜨는 것이다.
- WebMvcTest와 동일하게 `MockMvc`를 활용하여 디스패쳐 서블릿에 직접 요청을 꽂아 테스트한다.
### @SpringBootTest(Random Port) + HTTP Client
- 톰캣까지도 진짜로 띄우는 방식이다.
- 실제로 뜬 톰캣에 요청을 날리기 위해 스프링이 제공하는 `TestRestTemplate`를 활용할 수 있다.