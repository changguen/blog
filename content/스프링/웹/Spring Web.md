- 과거에는 WAS에 여러 서블릿을 끼워넣어 작동하는 구조였다.
- 하지만 이러다보니 서블릿마다 공통되는 코드들이 많이 존재했다.
- Spring Web에서는 이런 공통되는 부분들을 Front Controller 구조로 해결한다.
	- 디스패쳐 서블릿만 등록된다.
	- 모든 공통되는 로직들은 디스패쳐 서블릿과 협력 객체들에 의해 처리된다.
	- 개발자는 컨트롤러만 간단하게 구현하면 된다.
## Spring Web 동작 순서
![[Pasted image 20251119192823.png]]
1. WAS가 HTTP 요청을 받는다.
2. 여러 Filter들을 거쳐 DispatcherSerlvet까지 요청이 전달된다.
3. DispatcherServlet은 여러 HandlerMapping들을 확인하며 해당 요청을 처리할 수 있는 매핑이 존재하는지 확인한다.
	- ex) 애노테이션 기반 매핑은 `RequestMappingHandlerMapping`를 통해 확인된다.
4. HandlerMapping을 찾았다면 해당 HandlerMapping의 Handler를 찾는다.
5. Handler를 실행시킬 수 있는 HandlerAdapter를 찾는다.
	- ex) 애노테이션 기반 핸들러는 `RequestMappingHandlerAdapter`를 통해 실행된다.
6. 해당 HandlerAdapter를 실행시킨다.
	-  `RequestMappingHandlerAdapter`의 경우(애노테이션 기반) ArgumentResolver를 통해 핸들러 실행에 필요한 데이터를 세팅한다.
		- ex) `PathVariableMethodArgumentResolver`는 `@PathVariable`을 처리한다.
7. 모든 준비가 되었고, HandlerAdapter가 핸들러를 실행시킨다.
8. **핸들러 실행 및 응답** (서비스-리포지토리 등의 개발자 코드 실행)
9. 응답에 따라 ViewResolver나 HttpMessageConverter에 의하여 응답이 변환된다.
	- `ViewResolver` : 타임리프등을 사용했을 때 뷰를 렌더링
	- `HttpMessageConverter` : JSON 응답등을 생성 (`MappingJackson2HttpMessageConverter` 등)
10. 해당 응답을 DispatcherSerlvet으로 전달, 클라이언트에 응답한다.
