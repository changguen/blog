- **스프링에서 비동기 작업을 실행하는 인터페이스이다.**
- 자바의 [[ExecutorService]] 왜 매우 유사하다. (이를 스프링 환경에 맞게 추상화함)
- 기본적으로는 `SimpleAsyncTaskExecutor`가 구현되어 있다.
- 해당 구현체는 매번 새로운 스레드를 생성하여 작동하므로 비효율적이며, 위험하다.
- 따라서 사용할 때에는 다른 구현체를 설정하여 빈으로 등록하는 것이 권장된다.
- [[TaskDecorator]]를 통해서 실행 전후에 추가 작업을 해줄 수 있다.
## 스레드풀 Executor
```java
@Bean(name = "threadPoolTaskExecutor")  
public AsyncTaskExecutor threadPoolTaskExecutor() {  
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();  
    executor.setCorePoolSize(5);  
    executor.setMaxPoolSize(10);  
    executor.setQueueCapacity(25);  
    executor.initialize();  
    return executor;  
}
```