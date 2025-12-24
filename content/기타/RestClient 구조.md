- `RestClient` 는 내부적으로 `ClientHttpRequestFactory` 를 지니고 있으며, 해당 팩토리의 구현에 따라 요청 방식이 달라진다. (전략 패턴)
- `ClientHttpRequestFactory` 는 `ClientHttpRequest` 를 생성하게 된다.

```java
// DefaultRestClient 내부
private ClientHttpRequest createRequest(URI uri) throws IOException {
	ClientHttpRequestFactory factory;
	// ... RequestFactory 세팅 작업
	ClientHttpRequest request = factory.createRequest(uri, this.httpMethod);
	return request;
}
```

- `ClientHttpRequest`가 통신의 핵심이며, 이를 통해 통신하게 된다.
- 통신의 결과로는 `ClientHttpResponse` 를 받게 된다.

```java
// SimpleClientHttpRequest 내부
protected ClientHttpResponse executeInternal(HttpHeaders headers, Body body) throws IOException {
	if (this.connection.getDoOutput()) {
		long contentLength = headers.getContentLength();
		if (contentLength >= 0) {
			this.connection.setFixedLengthStreamingMode(contentLength);
		}
		else {
			this.connection.setChunkedStreamingMode(this.chunkSize);
		}
	}

	addHeaders(this.connection, headers);

	this.connection.connect();

	if (this.connection.getDoOutput() && body != null) {
		try (OutputStream os = this.connection.getOutputStream()) {
			body.writeTo(os);
		}
	}
	else {
		this.connection.getResponseCode();
	}
	return new SimpleClientHttpResponse(this.connection);
}
```

- 이를 `RestClient`가 적절하게 파싱하여 리턴해준다.

## 커스터마이징

---

- `RestClient` 는 내부의 `ClientHttpRequestFactory` 를 바꿔낌으로써 커스터마이징 포인트를 열어두었다.
- 기본 전략은 `SimpleClientHttpRequestFactory` 이며 커스터마이징할 것이 많이 없다. 타임아웃 정도?
- 다른 대표적 전략은 `HttpComponentsClientHttpRequestFactory` 인데, 커넥션 풀이나 프록시 설정등의 커스터마이징이 가능하다.

```java
ConnectionConfig connectionConfig = ConnectionConfig.custom()
        .setConnectTimeout(Timeout.ofSeconds(1))
        .setSocketTimeout(Timeout.ofSeconds(1))
        .build();

RequestConfig requestConfig = RequestConfig.custom()
        .setConnectionRequestTimeout(Timeout.ofSeconds(1))
        .setResponseTimeout(Timeout.ofSeconds(1))
        .build();

PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
connectionManager.setDefaultMaxPerRoute(150);
connectionManager.setMaxPerRoute(new HttpRoute(new HttpHost("localhost", 8080)), 10);
connectionManager.setMaxTotal(150);
connectionManager.setDefaultConnectionConfig(connectionConfig);

CloseableHttpClient httpClient = HttpClientBuilder.create()
        .setConnectionManager(connectionManager)
        .setDefaultRequestConfig(requestConfig)
        .build();

HttpComponentsClientHttpRequestFactory requestFactory =
        new HttpComponentsClientHttpRequestFactory(httpClient);

this.restClient = RestClient.builder()
        .baseUrl(osrmUrl)
        .requestFactory(requestFactory)
        .build();
```