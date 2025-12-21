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
	- 모든 `JobParameter`가 식별에 기여하는 것은 아니다.
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
- [[스프링 배치 Job 구성하기|구성 방법]]