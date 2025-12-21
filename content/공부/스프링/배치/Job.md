> 전체 배치 프로세스의 가장 상위 개념, [[Step|Step]]들의 논리적 묶음

![[Pasted image 20251220143556.png]]
- **JobInstance**
	- `Job`의 **논리적인 실행 단위**
	- 예를 들어, 매일 실행되는 Job의 경우, 5월 17일에는 1개의 `JobInstance`가 존재하게 된다.
	- 한번 실행한 `JobInstance`를 재실행 한다는 것은, 이전 실행의 상태를 사용한다는 것이며,
	  새로운 `JobInstance`를 실행한다는 것은 처음부터 시작한다는 것이다.
	- 각 `JobInstance`는 락 문제를 제외하고는 독립적이다.
	  따라서 스케줄러는 한 `Job`에 대한 여러 `JobInstance`를 동시에 실행시킬 수도 있다.
	- 동일한 `JobInstance`를 동시에 실행시키는 것은 `JobExecutionAlreadyRunningException` 예외를 발생시킨다.
- **JobParameters**
	- 각 **`JobInstance`를 구별하는 식별자**이다.
	- 또한 작업에 필요한 데이터를 전달하는 인자의 역할도 수행한다.
	- 즉, 'JobInstance = Job + JobParameters'
	```java
	public record JobParameters(Set<JobParameter<?>> parameters)
	```
	```java
	public record JobParameter<T>(String name, T value, Class<T> type, boolean identifying)
	```
	- 모든 `JobParameter`가 식별에 기여하는 것은 아니다. (identifying 참고)
- **JobExecution**
	- 하나의 `JobInstance`에 대한 **물리적인 실행 시도**
	- `JobInstance`는 `JobExecution`이 성공적으로 끝나지 않으면 완료된 것으로 간주되지 않는다.
	- 하나의 인스턴스에 대하여 실행 시도를 했다가, 실패했을 경우 또 시도할 수 있다.
	- 다음과 같은 구성 요소를 갖는다.
		- Status : 실행/성공/종료 상태
		- startTime
		- endTime
		- createTime
		- [[ExecutionContext|executionContext]] : 실행 간에 지속되어야 하는 데이터
		- faliureExceptions : 실행 중 발생한 예외 목록
- 참고 : 구현체는 SimpleJob이다. [[스프링 배치 Flow|FlowJob]]도 있다.
## Job 구성 방법
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
## JobRepository 구성 방법
- [공식 문서](https://docs.spring.io/spring-batch/reference/job/configuring-repository.html)
- [[Job|JobExecution]], [[Step|StepExecution]]과 같은 도메인 객체의 기본 CRUD 작업에 사용된다.
- 가장 간단한 구현체는 `ResourcelessJobRepository` 이며, 아무런 메타데이터를 저장하지 않는다.
	- 당연하게 [[ExecutionContext|ExecutionContext]]를 지원하지도 않는다.
	- **기본적으로 `@EnableBatchProcessing` 또는 `DefaultBatchConfiguration`을 사용할 때 제공된다.**
- 데이터베이스 기반의 구현체는 JDBC 기반과 MongoDB 기반이 제공된다.
- JDBC 기반으로 구성하기 위해서는 `@EnableJdbcJobRepository`를 사용한다.
```java
@EnableJdbcJobRepository(
		dataSourceRef = "batchDataSource",
		transactionManagerRef = "batchTransactionManager",
		tablePrefix = "BATCH_",
		maxVarCharLength = 1000,
		isolationLevelForCreate = "SERIALIZABLE")
```
- 위 방법으로 구성할 수 없는 경우, `JdbcJobRepositoryFactoryBean`을 사용하거나, `SimpleJobRepository`를 사용해야 한다.