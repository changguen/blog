# 클라이언트 요청 수신

- WAS에 의해 수신된다.
- WAS는 애플리케이션에 설정된 매핑에 따라 적절한 서블릿으로 해당 요청을 보낸다.
- 하지만 Spring Web MVC에서는 모든 요청을 `DispatcherServlet` 로만 보낸다.

# HandlerMapping 찾기

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for(HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }

    return null;
}
```

- 디스패쳐 서블릿 초기화시 같이 초기화된다.
- 모든 `HandlerMapping` 들을 뒤지며 현재 요청을 처리할 수 있는 `HandlerMapping` 을 찾는다.

## HandlerMapping

- 요즘은 보통 `@RequestMapping` 을 통해서 매핑을 만들기 때문에, `RequestMappingHandlerMapping` 이 선택된다.
- 해당 매핑에는 컨트롤러와 메서드 정보등이 등록되어 있다.
- 찾은 `HandlerMapping` 을 통해 `Handler` 를 반환한다.

# HandlerAdapter 찾기

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        for(HandlerAdapter adapter : this.handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }

    throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

- 디스패쳐 서블릿 초기화시 같이 초기화된다.
- 모든 `HandlerAdapter` 들을 뒤지며 현재 `Handler` 를 처리할 수 있는 `HandlerAdapter` 를 찾는다.

## HandlerAdapter

- 요즘은 `@Controller` 로 하지만, 옛날에는 핸들러를 구현할 수 있는 방법이 많았다.
- 여러가지 핸들러를 일관된 방식으로 호출할 수 있도록 해당 핸들러를 실행할 수 있는 어댑터가 존재한다.
- `@RequestMapping` 과 자주 사용되는 것은 `RequestMappingHandlerAdapter` 이다.

### RequestMappingHandlerAdapter

- 애노테이션 기반 핸들러를 실행한다.
- 만약 `@ResponseBody` 가 사용되었다면
    - `HttpMessageConverter` 를 통해 반환값을 변환하여 응답한다.
    - 비어있는 `ModelAndView` 를 반환한다.
- 만약 `@ResponseBody` 가 없었다면
    - `ModelAndView` 를 생성하여 반환한다.

### HttpMessageConverter

- 반환값을 기준으로 적절한 구현체를 찾는다.
    - 반환값이 `ResponseEntity` 이거나, 객체라면 `MappingJackson2HttpMessageConverter` 가 사용된다.
    - 반환값이 문자열이라면 `StringHttpMessageConverter` 가 사용된다.
- 각 구현체가 적절한 방법으로 반환값을 `HttpMessage`로 변환한다.

# 핸들러 실행

- 핸들러 어댑터가 해당 핸들러를 실행한다.
- 이 부분이 우리가 구현하는 부분이다.

# 뷰 렌더링

- `ModelAndView` 을 반환했다면 `ViewResolver` 를 통해 뷰를 렌더링한다.
- `ModelAndView` 가 없다면(null) 뷰 렌더링 과정은 생략된다.