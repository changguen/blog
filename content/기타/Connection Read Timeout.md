- `ClientHttpRequestFactory` 를 통해 타임아웃을 설정해줄 수 있다.
- 타임아웃 말고도 기타 잡다한 기능이 많아서, 보통 기존 구현체를 통해서 세팅해주게 된다.
- 여러 구현체들이 있는데, 기본 동작은 같으며 커넥션 관리 방식 / 성능 최적화 여부에 따라 달라진다.
    - `SimpleClientHttpRequestFactory` : 커넥션 풀도 없는 가장 간단한 형태
    - `HttpComponentsClientHttpRequestFactory` : 커넥션 풀을 포함한 여러 설정이 가능
    - `JettyClientHttpRequestFactory` : 비동기로 요청
    - [https://chatgpt.com/share/68ce67db-9d58-800f-985f-c59331b5c8ae](https://chatgpt.com/share/68ce67db-9d58-800f-985f-c59331b5c8ae)
- 내가 사용한 것은 `SimpleClientHttpRequestFactory` 이다.
    - 내가 원하는 것은 복잡한 성능 최적화가 아니기 때문이다.
    - 추후에 성능 최적화가 필요하다면 그 때 다른 구현체를 살펴볼 것 같다.

```java
public OsrmWalkingRouteService(final String osrmUrl) {
    final var requestFactory = new SimpleClientHttpRequestFactory();
    requestFactory.setConnectTimeout(Duration.ofSeconds(1));
    requestFactory.setReadTimeout(Duration.ofSeconds(2));

    this.restClient = RestClient.builder()
            .requestFactory(requestFactory)
            .baseUrl(osrmUrl)
            .build();
}
```