## CORS 필터

- 브라우저의 pre-flight `OPTIONS` 요청에 대한 응답을 설정한다.
- `CorsConfigurationSource` 타입 빈을 등록하거나, `SecurityFilterChain` 설정시 추가해주면 된다.
    
    ```java
    CorsConfiguration config = new CorsConfiguration();
    // 허용 Origin (와일드카드 패턴 사용 가능)
    config.setAllowedOriginPatterns(List.of(
        "https://frontend.example.com",
        "http://localhost:8080"
    ));
    
    // 허용 HTTP 메서드
    config.setAllowedMethods(List.of(
        "GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"
    ));
    
    // 브라우저가 보낼 수 있는 요청 헤더
    config.setAllowedHeaders(List.of(
        "Authorization",
        "Content-Type",
        "X-Requested-With"
    ));
    
    // 응답에서 브라우저가 읽을 수 있게 노출할 헤더
    config.setExposedHeaders(List.of(
        "Authorization",
        "Location"
    ));
    
    // 쿠키(JSESSIONID)·Authorization 헤더 전송 허용
    config.setAllowCredentials(true);
    
    // Pre-flight 결과 캐시 시간
    config.setMaxAge(Duration.ofHours(1));
    
    // Chrome PNA(Private Network Access) 허용 (옵션)
    // config.setAllowPrivateNetwork(true);
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
    ```
    

## CSRF 필터

- CSRF 공격 정리
    - 발생 이유 :브라우저가 자동으로 세션ID가 담긴 쿠키를 요청에 포함시키기 때문에 발생한다.
    - 사용자는 로그인 후에 세션 쿠키를 받는다.
    - 사용자가 다른 사이트에 접속한다. (악성 사이트)
    - 악성 사이트는 악의적인 요청을 보낼 수 있고, 쿠키가 같이 보내지기 때문에 서버 수준에서는 공격자와 사용자를 구분할 수 없다. (사용자인 것으로 인지된다.)
    - 방어 방법
        - 세션 쿠키 말고도 CSRF 토큰을 헤더에 포함해야 요청할 수 있도록 한다.
        - 악성 사이트가 요청을 보낼 때 세션 쿠키는 포함되더라도 CSRF 토큰은 포함할 수 없기 때문에(쿠키가 아니라서 자동포함X) 방어된다.
- CSRF 필터 사용 환경
    - 토큰 기반 인증에서는 요청시 토큰을 명시적으로 포함해야 하기 때문에 CSRF 공격 자체가 불가능하다. → 필터 OFF
    - 세션 기반 인증에서는 CSRF 필터를 구성해야 한다.
- CSRF 필터 동작 방식
    - 첫 참조시 `CsrfTokenRepository` 가 토큰을 발급하여 클라이언트에게 반환한다.
    - 다음 요청부터는 `X-XSRF-TOKEN` 헤더에 해당 토큰을 포함하여 요청하면, `CsrfFilter` 가 이를 통해 비교한다.
        - 예상되는 CSRF 토큰과 실제 넘어온 토큰을 비교한다.
- 설정 포인트
    - `CsrfTokenRepository`
        - 기본적으로 HttpSession에 예상 CSRF 토큰을 저장해놓는다. (`HttpSessionCsrfTokenRepository`)
        - `CookieCsrfTokenRepository` : 예상 CSRF 토큰도 클라이언트 쿠키로 보내버린다. 따라서 클라이언트는 쿠키/헤더 모두 CSRF 토큰을 보내고 서버에서는 이 두개가 일치하는지만 확인하면 된다. STATELESS 하다.
            - 클라이언트가 쿠키를 읽어서 헤더에 명시적으로 포함해야할 필요가 있다. → `withHttpOnlyFalse()` 옵션을 통해서 JS에서 쿠키를 읽을 수 있도록 해야 한다.
            - 또한 해당 방식을 지원하기 위해서는 CSRF 토큰을 쿠키에 포함해줘야 한다. → 커스텀 핸들러를 작성해야 한다.
                
                ```java
                @Bean
                SecurityFilterChain api(HttpSecurity http) throws Exception {
                    http
                      .csrf(csrf -> csrf
                          .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                          .ignoringRequestMatchers("/public/**")
                          .csrfTokenRequestHandler(new SpaCsrfTokenRequestHandler())
                      )
                      .authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
                    return http.build();
                }
                ```
                
                ```java
                public class SpaCsrfTokenRequestHandler implements CsrfTokenRequestHandler {
                
                  private final CsrfTokenRequestHandler plain = new CsrfTokenRequestAttributeHandler();
                  private final CsrfTokenRequestHandler xor = new XorCsrfTokenRequestAttributeHandler();
                
                  /*
                  CSRF 관련 문서에서 권장하는 방식
                  https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html
                   */
                  @Override
                  public void handle(HttpServletRequest req, HttpServletResponse res, Supplier<CsrfToken> deferred) {
                    xor.handle(req, res, deferred);
                    deferred.get();
                  }
                
                  @Override
                  public String resolveCsrfTokenValue(HttpServletRequest req, CsrfToken token) {
                    return (req.getHeader(token.getHeaderName()) != null ? plain : xor)
                        .resolveCsrfTokenValue(req, token);
                  }
                }
                ```