- 스프링에서 내부의 이벤트를 동기/비동기 방식으로 처리할 수 있게 해주는 애노테이션이다.
- 기본적으로는 동기적으로 작동한다. (같은 스레드에서 실행된다.)
### 이벤트 발행
```java
@Service
public class UserService {
    
    private final ApplicationEventPublisher eventPublisher;
    
    public UserService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }
    
    public void registerUser(String username) {
        // 사용자 등록 로직
        eventPublisher.publishEvent(new UserRegisteredEvent(username));
    }
}
```
### 이벤트 리스너
```java
@Component
public class UserEventListener {
    
    @EventListener
    public void handleUserRegistered(UserRegisteredEvent event) {
    }
}
```
### 리스닝하는 여러 방식들
- 비동기
	- 기본적으로 동기적으로 작동하기 때문에, 이벤트리스너의 작업이 모두 끝날 때까지 블로킹된다.
	- 논블로킹하려면 [[@Async]] 도 같이 사용해주면 된다.
- 트랜잭션 연동
	- 기본적으로 이벤트 발행자의 트랜잭션 상태에 상관없이 작동하기 때문에, 커밋되기 전에 조회하는 등의 문제가 발생할 수 있다.
	- 커밋 상태에 따라 작동하려면 `@TransactionalEventListener`를 사용하고 옵션을 주면 된다.