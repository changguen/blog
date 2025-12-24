### HTTP Client API (java.net.http)
기존에는 HTTP 통신을 위하여 복잡한 `HttpURLConnection`를 쓰거나, 외부 라이브러리에 의존해야 했다.
```java
URL url = new URL("https://api.example.com/users");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setRequestProperty("Content-Type", "application/json");

int responseCode = conn.getResponseCode();
BufferedReader in = new BufferedReader(
    new InputStreamReader(conn.getInputStream()));
String inputLine;
StringBuilder response = new StringBuilder();
while ((inputLine = in.readLine()) != null) {
    response.append(inputLine);
}
in.close();
conn.disconnect();
```
**HTTP Client API**는 표준 HTTP 클라이언트를 제공하고, 네이티브로도 표현력 있는 코드를 작성하는 것을 목표로 한다.
```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(
    request, 
    HttpResponse.BodyHandlers.ofString()
);
String body = response.body();
```
비동기/웹소컷 통신도 제공하며, HTTP/2 도 지원한다.