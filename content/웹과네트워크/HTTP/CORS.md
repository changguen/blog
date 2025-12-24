- **브라우저가 자신의 출처가 아닌 다른 브라우저로부터 자원을 로딩하는 것을 허용하는 HTTP 헤더 기반 메커니즘이다.**
    - 그냥 다 허용하면 다음과 같은 문제가 발생한다.
    - 은행 앱의 쿠키를 들고 악성 사이트에 접속했는데, 악성 사이트가 몰래 해당 쿠키를 다른 출처로 보내버릴 수 있다.
    - 따라서 기본적으로 이는 막혀 있으며, 특정한 경우에만 CORS를 통해 허용하는 것이다.
    - 정리하면, CORS는 브라우저 사용자가 자기도 모르게 민감한 인증 정보를 포함한 요청을 다른 출처로 보내는 것을 막는 것을 일부 허용하는 것이다.
- `https://domain-a.com` 에서 제공되는 프론트엔드 JavaScript 코드가 `fetch()`를 사용하여 `https://domain-b.com/data.json` 에 요청하는 경우가 이에 해당한다.

![[CORS_aa.svg]]

- CORS는 서버가 웹 브라우저에서 해당 정보를 읽는 것이 허용된 출처를 설명할 수 있도록 새로운 HTTP 헤더를 추가함으로써 동작한다.
- **부수효과를 일으킬 수 있는 HTTP 요청 방법**에 대하여, 서버에 먼저 preflight한 후, 서버에게 승인을 받고 실제 요청을 보내도록 한다.

# Simple Request

- Simple Request는 부수효과를 일으키지 않는다.
- 따라서 preflight를 할 필요도 없다!
- 부수효과를 일으키지 않는 HTTP 요청 방법은 다음과 같다.
    - `GET`
    - `HEAD`
    - `POST`
        - Content-Type이 다음중 하나여야 한다.
        - `application/x-www-form-urlencoded`
        - `multipart/form-data`
        - `text/plain`
- 이들은 CORS와 상관없이 그냥 서버에 요청한다.

# Preflight Request

- 위 Simple Request가 아닌 경우에 보내는 사전 요청이다.
- `OPTIONS` 메서드를 사용한다.
- Preflight Request의 시나리오는 다음과 같다.

```
**OPTIONS** /doc HTTP/1.1
Host: bar.other
Origin: https://foo.example
**Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type,x-pingother**

HTTP/1.1 204 No Content
**Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400**
```

- `Access-Control-Request-Method` : 허용된다고 하면 보낼 실제 메서드
- `Access-Control-Request-Headers` : 허용된다고 하면 보낼 실제 헤더
- `Access-Control-Allow-Origin` : 서버가 허용하는 Origin
- `Access-Control-Allow-Methods` : 서버가 허용하는 메서드
- `Access-Control-Allow-Headers` : 서버가 허용하는 헤더
- `Access-Control-Max-Age` : 현재 preflight에 대한 유효 시간 (초)
- 서버는 preflight 요청 헤더 정보를 바탕으로 해당 요청을 허용할 것인지 결정한다. 결정한 후에 응답 헤더를 세팅하는 것이다.
- **preflight가 보내지고, 응답을 받았을 때 실제 요청이 허용된다면 실제 요청을 자동으로 보낸다.**

# 자격 증명

- **기본적으로 위 방법으로는 실제 요청에 자격 증명을 포함할 수 없다.**
    - 자격 증명이란 쿠키, 인증헤더 등을 말한다.
- 서버는 preflight 응답 헤더에 실제 요청에서 자격 증명을 포함해도 된다고 명시해야 한다.

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
**Access-Control-Allow-Credentials: true**
Access-Control-Max-Age: 86400
```

- 또한 `Access-Control-Allow-Origin` 를 정확히 명시해야 한다.
    - 기존에는 `*`이 허용되었는데, 자격 증명을 포함하도록 하면 Origin을 정확히 명시하도록 해야 한다.