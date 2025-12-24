기존 `ThreadLocal`의 대안이다.
- 기존에는 remove를 까먹으면 메모리누수로 이어지지만, Scoped Value는 범위를 넘어가면 알아서 제거된다.
- 기존 값은 언제든 변경 가능하여 안전하지 않다.
- [[가상스레드]]에서는 성능 문제가 존재한다. (가상스레드는 기본적으로 매우 많이 스레드를 찍어내는 방식으로 작동하기 때문에, 크기가 매우 커진다.)
**ScopedValue**는 이 문제들을 해결한다.
- 값이 어디까지 유효한지 API 수준에서 명확하게 나타내고, 범위를 벗어나면 자동 제거된다.
- 불변이다.
- 값을 복사하는게 아니고, 참조만 복사하기 때문에 메모리 사용량이 적다.
## 인증 예제
```java
// 인증 후 User가 담기는 컨텍스트
public class UserContext {
    public static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

    public static User get() {
        return CURRENT_USER.orElseThrow(() -> new IllegalStateException("인증된 사용자가 없습니다."));
    }
}

// 인증 필터
public class AuthenticationFilter extends OncePerRequestFilter {

    private final JwtProvider jwtProvider;
    private final UserRepository userRepository;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {

        Long userId = jwtProvider.resolveAndValidateToken(request);
        if (userId == null) {
	        filterChain.doFilter(request, response);
	        return;
        }
        User user = userRepository.findById(userId).orElseThrow(() -> new IllegalArgumentException("User not found"));
		ScopedValue.where(UserContext.CURRENT_USER, user) // 이후 컨트롤러 등에서 USER에 접근할 수 있게 된다.
                .run(() -> filterChain.doFilter(request, response));
    }
}

// 컨트롤러 코드에서 사용하기
@GetMapping("/me")
public String getMyInfo() {
	User user = UserContext.get(); 
	return "안녕하세요, " + user.getName() + "님!";
}
```