- `JobBuilder`를 통해 구성한다.
- 기본적으로는 [[Step|Step]]을 끼워넣어 구성하면 된다.
- [[스프링 배치 Flow|Flow]]를 끼워넣을 수도 있다. (병렬과 스플릿 개념이 추가된다.)
- [[Listener|리스너]]를 끼워넣을 수도 있다.
- `JobExecutionDecider`를 끼워넣어서 흐름을 제어할 수 있다. (=Decision)
- `JobParametersValidator`를 끼워넣어서 검증할 수 있다. 
- **재시작 가능성**
	- [[Job|JobInstance]]에 대해 [[Job|JobExecution]]이 이미 있는 경우, 다음 JobExecution은 **재시도로 간주된다.**
	- 이상적으로는 모든 작업이 중단된 지점부터 시작되어야 하지만, 불가능한 시나리오도 있다.
	  -> `preventRestart()` 구성을 통해 재시작을 막을 수 있으며, 강제로 재시작하면 `JobRestartException` 예외가 터진다.
```java
@Bean
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .build();
}
```