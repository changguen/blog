## Cookie

- `Set-Cookie` 헤더를 통해 브라우저 수준에서 자동으로 쿠키가 생성된다.
- 이후에는 자동으로 요청에 쿠키가 포함된다.

## Bearer Token

- 응답 json을 통해 ‘Beaer 123123’ 과 같은 형식이 반환된다.
- 이후 클라이언트가 수동으로 직접 Authorization 헤더에 해당 값을 포함한다.

## 보안성 비교

- CSRF(Cross-site Request Forgery)
    - 쿠키는 자동으로 포함되기 때문에 CSRF에 취약하다.
    - Bearer는 수동으로 포함되기 때문에 CSRF에 강하다.
- XSS(Cross-site Scripting)
    - 쿠키에 HttpOnly 옵션을 붙이면 JS에서 접근이 불가능하기 때문에 XSS에 강하다.
    - Bearer는 JS의 로컬스토리지 등에 저장되기 때문에 XSS에 취약하다.