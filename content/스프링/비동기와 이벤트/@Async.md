- 스프링은 기본적으로 전체가 동기적으로 작동한다.
- 특정 메서드를 비동기적으로 사용하도록 하기 위해서는 `@Async` 애노테이션을 사용한다.
- AOP로 작동하며, 따라서 자기 자신을 호출하면 작동하지 않는다.
- 비동기 메서드의 예외는 호출자로 전파되지 않는다.
- 원하는 [[AsyncTaskExecutor]]를 사용하도록 할 수 있다.
## 사용 방법
1. Async를 켜준다.
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    // 기본 SimpleAsyncTaskExecutor 사용
}
```
2. 원하는 메서드에 적용한다.
```java
@Service
public class EmailService {
    
    @Async
    public void sendEmail(String to, String content) {
        // 별도 스레드에서 실행됨
    }
}
```
## 반환타입
- void : 반환하지 않는다.
- `Future<T>` : 결과를 나중에 가져올 수 있음
- `CompletableFuture<T>` : 더 유연한 비동기 응답
## 작동 방식
1. AOP로 메서드 호출을 가로챈다.
2. `@Async` 애노테이션의 [[AsyncTaskExecutor]] 이름을 확인하고 가져온다. (없다면 기본 구현체 사용)
3. Executor에게 해당 메서드 호출을 제출한다.