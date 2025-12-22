## 순차적 흐름
![[Pasted image 20251221233413.png]]
- 기본적인 흐름이다.
- 이 흐름에서는 다음 가능성만이 존재한다.
	- 한 Step이 성공하면 다음 Step을 실행한다.
	- 마지막 Step이 성공하면 Job이 성공한다.
	- 한 Step이 실패하면 Job이 실패한다.
- **대부분의 경우 이 흐름으로 충분하다.**
```java
@Bean
public Job job(JobRepository jobRepository, Step stepA, Step stepB, Step stepC) {
	return new JobBuilder("job", jobRepository)
				.start(stepA)
				.next(stepB)
				.next(stepC)
				.build();
}
```
## 조건부 흐름
![[Pasted image 20251221235533.png]]
- 성공시 B Step, 실패시 C Step을 실행하는 흐름이다.
  이것외에도 `ExitStatus`를 기반으로 적절하게 라우팅할 수 있다.
```java
@Bean
public Job job(JobRepository jobRepository, Step stepA, Step stepB, Step stepC) {
	return new JobBuilder("job", jobRepository)
				.start(stepA)
				.on("*").to(stepB)
				.from(stepA).on("FAILED").to(stepC)
				.end()
				.build();
}
```
- `on()` 메서드에 들어가는 문자열 값은 `ExitStatus` 이름 기반의 패턴매칭이다.
	- `*` : 0개 이상 문자 일치
	- `?` : 1개 문자 일치
- 이 구성을 자동으로 가장 구체적인 것부터 정렬한다.
- **주의! `ExitStatus` 와 `BatchStatus`는 다르다.**
  `ExitStatus`는 열거형이 아니며, 따라서 [[Listener|StepExecutionListener]]를 통해서 커스터마이징한 `ExitStatus`를 응답하고, 이를 참조할 수 있다.
- 플로우를 더 프로그래매틱하게 결정하기 위해서 `JobExecutionDecider`를 구현한 후, 이를 설정에 끼워넣을 수도 있다.
### 조건부 흐름에서의 Job의 BatchStatus와 ExitStatus 결정
- [[Step]]의 `BatchStatus`, `ExitStatus`는 명료하다. 코드에 의해 결정된다.
  하지만, [[Job]]의 그것은 설정에 따라 달라진다.
- **기본 (별도의 전이 설정 X)**
	- Job은 **마지막** Step의 결과를 따른다.
	- 이 케이스로 보통 충분하지만, 아닌 경우도 존재한다.
- **미리 COMPLETED**
	- 조건부 흐름에서 어떠한 상황에서 정상적으로 끝내고 싶은 경우이다.
	  ex) 데이터가 하나도 없으면 그냥 COMPLETED로 종료
	- 이 경우, 해당 Step까지 실행하고, Job은 COMPLETED가 된다.
	- 아래 예시에서 step2가 실패하면 바로 Job이 끝나고 COMPLETED가 된다. (재시도 불가능)
```java
@Bean
public Job job(JobRepository jobRepository, Step step1, Step step2, Step step3) {
	return new JobBuilder("job", jobRepository)
				.start(step1)
				.next(step2)
				.on("FAILED").end()
				.from(step2).on("*").to(step3)
				.end()
				.build();
}
```
- **미리 FAILED**
	- 조건부 흐름에서 어떠한 상황에서 비정상적으로 끝내고 싶은 경우이다.
	  ex) 데이터 검증에 실패하면 그냥 FAILED로 종료
	- 이 경우, 해당 Step은 `BatchStatus=FAILED, ExitStatus=EARLY TERMINATION`이다.
	- 아래 예시에서 step2가 실패하면 바로 Job이 끝나고 FAILED가 된다. (재시도 가능)
```java
@Bean
public Job job(JobRepository jobRepository, Step step1, Step step2, Step step3) {
	return new JobBuilder("job", jobRepository)
			.start(step1)
			.next(step2).on("FAILED").fail()
			.from(step2).on("*").to(step3)
			.end()
			.build();
}
```
- **STOPPED**
	- 조건부 흐름에서 어떠한 상황에 Job을 일단 중지하고 싶은 경우이다.
	  ex) 토큰 무료 사용량이 아슬아슬하면 일단 STOPPED로 종료
	- 아래 예시에서 step1이 성공하면 Job이 중지되며, 재시작시 step2부터 시작한다.
```java
@Bean
public Job job(JobRepository jobRepository, Step step1, Step step2) {
	return new JobBuilder("job", jobRepository)
			.start(step1).on("COMPLETED").stopAndRestart(step2)
			.end()
			.build();
}
```
## [[Flow|병렬 흐름]]
- 지금까지의 흐름은 선형적으로 한번에 하나씩 실행되는 것이었지만, 병렬 흐름으로도 구성할 수 있다.
- 이를 위해서 `Flow`라는 것을 도입한다.
- `Flow`는 사실 선형적 흐름 그 자체이다. 지금까지 만든 것들이 사실 `Flow`이다.
- `Flow`를 활용하면 다음처럼 병렬 흐름 구성이 가능하다.
![[Pasted image 20251222102540.png]]
## Job을 Step 처럼 외부화
- 다음처럼 구성하여 Job을 하나의 Step처럼 구성하고, 다른 Job에 활용할 수 있다.
```java
@Bean
public Step jobStep(JobRepository jobRepository, JobLauncher jobLauncher, Job otherJob, JobParametersExtractor jobParametersExtractor) {
    return new StepBuilder("jobStep", jobRepository)
            .job(otherJob) // 실행할 다른 Job
            .launcher(jobLauncher)
            .parametersExtractor(jobParametersExtractor) // 부모 Job의 파라미터를 자식에게 전달하는 전략
            .build();
}
```
![[Pasted image 20251222103304.png]]