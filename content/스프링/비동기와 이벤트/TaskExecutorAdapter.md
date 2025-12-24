- 비동기 관련 실행기들은 다음 의존관계를 갖는다.
Executor <- TaskExecutor <- [[AsyncTaskExecutor]]
↑
[[ExecutorService]]
- 스프링은 `AsyncTaskExecutor`를 사용하여 [[@Async]]를 처리하게 된다.
- 하지만 `Executor`쪽으로만 구현된 기능을 사용하려면 `Executor`를 `AsyncTaskExecutor` 처럼 사용하도록 해야 한다.
- 이를 위해서 `TaskExecutorAdapter`를 사용한다.
- `TaskExecutorAdapter`는 `AsyncTaskExecutor`의 구현체이면서, 생성자로 `Executor`를 받기 때문에, 자바의 기능들이 지원된다.
```java
ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();
AsyncTaskExecutor asyncTaskExecutor = new TaskExecutorAdapter(executorService);
```