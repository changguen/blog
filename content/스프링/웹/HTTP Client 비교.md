### RestTemplate
- Spring 3부터 있었던 오래된 방식이다.
- 그만큼 자료가 많으나, 현재 유지보수 모드로 돌아가 더 이상의 기능 추가는 없다.
### WebClient
- 비동기/논블로킹 HTTP Client이다.
- 이벤트 루프 기반이며, 적은 스레드로도 많은 요청을 처리할 수 있다.
- MVC 프로젝트에서 사용하기 위해서는 webflux 의존성을 추가해야 한다. 이런 경우에는 동작 방식만 다른 `RestClient`를 사용하자.
### RestClient
- 동기 HTTP Client 이다.
- `WebClient`의 API를 따라했다.
### HTTP Interface
- 가장 최신 방식이며 스프링7에서 지원 기능이 더 많아졌다.
- `OpenFeign` 처럼 선언적으로 작동한다.
- 구현체는 결국 WebClient / RestClient 이다. (즉, 선언적 API의 껍데기가 하나 있을 뿐이다.)
- 인터페이스이다 보니, 구현체가 추후에 바뀌더라도 유연하게 대응할 수 있다.