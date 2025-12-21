![[Pasted image 20251220214019.png]]
- [[Job]]을 시작하거나, 재시작하고 중지하는 등, '운영'과 '제어'를 담당하는 최상위 인터페이스이다.
- 시작 시에 [[Job]]을 넘겨줘야 하는데, [[Job]]을 그대로 주입받아서 넘겨주어도 되고, 동적으로 [[Job]]을 바꿀 필요가 있다면 JobRegistry를 통해 동적으로 찾아서 넘겨주면 된다.
## 구성 방법
- **직접 구성할 일은 많이 없다. 최대한 자동 구성을 베이스로 가져가는 것이 좋다.**
- 가장 기본적인 구현체는 `TaskExecutorJobOperator`이다. 팩토리 빈을 통해 다음처럼 구성한다.
```java
@Bean
public JobOperatorFactoryBean jobOperator(JobRepository jobRepository) {
	JobOperatorFactoryBean jobOperatorFactoryBean = new JobOperatorFactoryBean();
	jobOperator.setJobRepository(jobRepository);
	return jobOperatorFactoryBean;
}
```
- 이는 동기적으로 작동하며, 비동기로 작동시키고 싶으면 [[AsyncTaskExecutor|TaskExecutor]]를 넘겨주어야 한다.
- 팩토리 빈으로 등록하여 구성하면 내부적으로 트랜잭션 처리가 적용된다.
## 우아한 종료
- 다음처럼 설정하면 프레임워크로 제어권이 반환되는 즉시 `BatchStatus.STOPPED`가 된다.
- StepExceution과 JobExecution에 대해 모두 동일하게 작용한다.
```java
Set<Long> executions = jobOperator.getRunningExecutions("sampleJob");
jobOperator.stop(executions.iterator().next());
```