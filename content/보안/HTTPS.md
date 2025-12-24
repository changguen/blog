## HTTPS 란?

- HTTP에 TLS 보안 계층이 추가된 것이다.
- TLS 보안 계층 덕분에 다음 장점을 누린다.
    - 암호화 : 중간에 데이터를 가로채더라도 읽지 못한다.
    - 인증 : 클라이언트가 접속하려는 서버가 위조된 서버가 아니라 신뢰할 수 있는 서버임을 보장한다.
    - 무결성 : 중간에 데이터가 변조된 것을 감지할 수 있다.

## TLS와 인증서

- TLS의 핵심은 CA가 서명한 인증서이다.
- 인증서는 다음과 같은 효과를 가진다.
    - CA가 서명했기 때문에 클라이언트가 서버를 믿을 수 있다.
    - 서버의 공개키가 포함되어 있기 때문에 이를 통해 클라이언트와 서버는 대칭키를 공유할 수 있다.
    - 이렇게 공유된 대칭키가 통신에 사용된다.
- 인증서에는 다음과 같은 정보가 담긴다.
    - **서버의 공개키** : 대칭키를 주고받기 위해 사용된다.
    - **발급 대상** : 이 인증서의 도메인을 확인한다.
    - **발급자** : 신뢰할 수 있는 CA가 발급했는지 확인한다.
    - **발급자의 서명** : 해당 인증서가 위변조되지 않음을 확인한다.
- TLS 통신 과정 (간략하게)
    1. **서버 - 인증서 → 클라이언트**
    2. **클라이언트가 해당 인증서를 해독한다.**
        - 클라이언트가 내장된 CA 목록에서 CA의 공개키를 찾아서 서명을 검증한다.
        - 인증서에 명시된 도메인이 현재 내가 접속하려는 도메인과 동일함을 확인한다.
    3. **서버 공개키를 통해 클라이언트와 서버가 대칭키를 나눠 갖는다.**
        - RSA 방식 (구닥다리)
            1. 클라이언트가 **이번 통신에서만 사용할** 랜덤 대칭키를 생성하고, 서버 공개키로 암호화한다.
            2. 클라이언트 - 암호화된 대칭키 → 서버
            3. 서버가 암호화된 대칭키를 자신의 비밀키로 복호화하여 대칭키를 양쪽 모두 소유하게 된다.
        - DHE/ECDHA 방식 (최신)
            1. 클라이언트와 서버가 각각 **이번 통신에서만 사용할** CLI와 SER이라는 비밀 숫자를 정한다.
            2. 클라이언트와 서버가 BASE 라는 공개된 내용을 공유한다.
            3. 클라이언트는 자신의 숫자로 BASE를 계산한다. (BASE ^ CLI)
            4. 서버는 자신의 숫자로 BASE를 계산한다. (BASE ^ SER)
            5. 이것을 서로 교환한다.
            6. 클라이언트는 받은 것을 자신의 숫자로 다시 한번 계산한다. (BASE ^ SER ^ CLI)
            7. 서버는 받은 것을 자신의 숫자로 다시 한번 계산한다. (BASE ^ CLI ^ SER)
            8. 둘 다 동일한 대칭키를 갖게 되었다.
        - 참고 : DHE에서 서버의 공개키는 중간자 공격을 막기 위해 통신 과정에서 사용된다. 대칭키를 만드는 계산 자체에는 들어가지 않는다.
        - 참고 : E는 일시적이라는 뜻의 Ephemeral이다.
        - 참고 : RSA는 대칭키를 생성하는 과정에서 서버의 공개/비밀키를 사용하며, 따라서 추후에 서버의 개인키를 탈취당하면 과거의 모든 내용을 읽을 수 있게 된다. (PFS 보장 X) 이에 반해 DHE/ECDHA는 PFS를 달성할 수 있다.
    4. **클라이언트 ↔ 데이터(대칭키로 암호화) ↔ 서버**

## 이렇게나 좋은 HTTPS 적용하기

- **인증서 발급받기**
    1. 서버 공개키/비밀키 만들기
    2. CSR(Certificate Signing Request) 작성하고 CA에 제출하기
    3. CA가 정보를 확인하고, 인증서를 발급
- **WS에 인증서 설치하기**
    1. WS에 인증서를 업로드한다.
    2. 443포트를 리스닝하도록 하고, SSL 관련 지시어를 설정한다.
    3. HTTP(80포트) 요청을 HTTPS(443포트)로 리다이렉트 시키도록 설정한다.
- 참고 : WAS에서 TLS까지 설정하면 관리가 복잡해지므로, 보통 리버스 프록시를 통해 Nginx등의 웹서버에 인증서를 설치한다.
- Nginx 설정 파일 예시
    
    ```
    # HTTP 요청을 HTTPS로 리다이렉트하는 서버 블록
    server {
        listen 80;
        server_name my-domain.com www.my-domain.com;
        return 301 https://$host$request_uri;
    }
    
    # HTTPS 요청을 처리하는 메인 서버 블록
    server {
        listen 443 ssl; # 443 포트에서 SSL 통신을 리스닝
        server_name my-domain.com www.my-domain.com;
    
        # 1. SSL 인증서 설정
        ssl_certificate /etc/nginx/ssl/fullchain.pem; # 합친 인증서 파일 경로
        ssl_certificate_key /etc/nginx/ssl/my-domain.key; # 개인키 파일 경로
    
        # 2. SSL 보안 강화 설정 (권장)
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    
        # HSTS (Strict Transport Security) 설정 (6개월)
        add_header Strict-Transport-Security "max-age=15768000; includeSubDomains" always;
    
        # 3. 애플리케이션 연결 (리버스 프록시)
        location / {
            proxy_pass http://localhost:8080; # Spring Boot 애플리케이션 주소
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    
        # 기타 로그 설정, 루트 디렉토리 등
        root /var/www/html;
        index index.html index.htm;
    }
    ```