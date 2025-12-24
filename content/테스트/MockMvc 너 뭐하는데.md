아 저요? 저는 MVC를 테스트 할 수 있도록 도와주는데요.

`DispatcherSerlvet`에 진짜 `HttpServletRequest` 를 꽂는게 아니고, 모킹을 한 `MockHttpServletRequest` 라는 객체를 사용해서 테스트해요.

그니까, HTTP 요청이 아니라 자바 문법으로 Request를 생성할 수 있도록 도와주고 그걸 `DispatcherServlet` 으로 꽂아주는거죠. 그 후에 `MockHttpServletResponse` 를 받아가지고 그걸로 또 자바 문법으로 검증할 수 있도록 도와줘요.

그러니까 쉽게 말하면 HTTP ↔ DispatcherSerlvet 간 통신을 자바 문법 ↔ DispatcherServlet 간 통신으로 바꿔주는 그런 통역기죠.