- HTTP 응답헤더의 `Connection` 은 TCP 연결을 유지할지를 뜻한다.

# HTTP 1.0 이하

- 기본적으로 매 HTTP 요청마다 TCP 연결을 새로 한다.
- `Connection : Keep-Alive` 를 통해서 이후 요청에서 TCP 연결을 유지할 수 있다.

# HTTP 1.1 이상

- 웹 세상이 복잡해지면서 요청 간격이 매우 짧아졌다.
- 매번 TCP 연결하는 비용이 TCP 연결을 유지하는 비용보다 커졌다.
- 기본적으로 첫 요청에만 TCP 연결하고, 이후에는 해당 연결을 계속 사용한다.
- `Connection : Close` 를 통해서 TCP 연결을 끊을 수 있다.