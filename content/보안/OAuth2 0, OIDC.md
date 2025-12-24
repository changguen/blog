> 우리 서비스를 사용하려는 클라이언트를 ‘사용자’, 우리 서비스를 ‘클라이언트’, OAuth 서버를 ‘구글’이라고 부르겠다.

실제 상황에서는 보통 각각 오너, 클라이언트, 서버 라고 부른다.
> 

## **OAuth**

**: 클라이언트가 사용자의 정보에 접근하기 위하여 구글에게 접근 권한을 달라고 하는 것**

### 준비 단계

- 클라이언트가 구글에 자신을 등록해야 한다.
- 이때 `Redirect URI`를 등록한다.
- 구글에서 인증을 마치고 사용자를 리다이렉트 시킬 위치이다.
- 등록을 마치면 `Client ID`, `Client Secret` 을 얻을 수 있다. 클라이언트는 이를 잘 저장해놓는다.

### 실제 동작

1. 사용자가 클라이언트의 ‘구글로 로그인하기’ 버튼을 클릭한다.
2. 클라이언트는 사용자의 브라우저를 구글로 보내야 한다.
    - 이때 쿼리스트링을 통해 몇가지 값을 더 보낼 수 있다.
        
        ```
        https://authorization-server.com/auth?
        response_type=code
        &client_id=29352735982374239857
        &redirect_uri=https://example-app.com/callback
        &scope=create+delete
        ```
        
    - `response_type` : 반드시 `code` 로 값을 설정해야한다.
    - `client_id` : 애플리케이션을 생성했을 때 발급받은 Client ID
    - `redirect_uri` : 애플리케이션을 생성할 때 등록한 Redirect URI
    - `scope` : **클라이언트가 부여받은 리소스 접근 권한**
3. 구글은 로그인 페이지를 사용자에게 제공하며, 사용자는 로그인 한다.
4. 구글은 `Authorization Code`를 발급하고, 사용자를 아까 등록했던 `Redirect URI` 로 해당 값을 포함하여 리다이렉트 시킨다.
5. 클라이언트는 구글에 `Authorization Code` 를 제출하고, `Access Token` 으로 교환한다.
    
    ```
    POST /oauth/token HTTP/1.1
    Host: authorization-server.com
    
    grant_type=authorization_code
    &code=xxxxxxxxxxx
    &redirect_uri=https://example-app.com/redirect
    &client_id=xxxxxxxxxx
    &client_secret=xxxxxxxxxx
    ```
    
    - `grant_type` : 항상 `authorization_code` 로 설정되어야 한다.
    - `code` : 발급받은 Authorization Code
    - `redirect_uri` : Redirect URI
    - `client_id` : Client ID
    - `client_secret` : RFC 표준상 필수는 아니지만, Client Secret이 발급된 경우 포함하여 요청
6. `Access Token` 을 잘 교환했다면 클라이언트는 사용자에게 로그인 성공 메시지를 보낸다.
7. 사용자가 클라이언트의 서비스를 사용 중, 구글과 연동된 서비스를 사용하고 싶어한다.
8. 클라이언트는 `Access Token` 으로 구글의 API를 호출하여 서비스를 제공한다.

## OpenID Connect

**: OAuth를 확장하여, 인증 기능을 추가하는 것이다.**

### 실제 동작

- 위 OAuth 동작에서, 2번 과정부터 조금 달라진다.
    - `scope` 에 `openid` 값을 추가한다.
- 이러면 구글은 클라이언트에게 `Access Token` 과 함께 **`ID Token`** 을 반환한다.
- `ID Token` 에는 사용자의 식별정보가 들어있다.
- 신규 식별정보라면 회원가입을 한다. : 클라이언트의 DB에 식별정보를 저장한다.
- 기존 식별정보라면 로그인을 한다.