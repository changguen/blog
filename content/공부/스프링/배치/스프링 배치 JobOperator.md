![[Pasted image 20251220214019.png]]
- Job을 시작하거나, 재시작하고 중지하는 등, '운영'과 '제어'를 담당하는 최상위 인터페이스이다.
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