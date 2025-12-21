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
## Job의 BatchStatus와 ExitStatus 결정
- [[Step]]의 `BatchStatus`, `ExitStatus`는 명료하다. 코드에 의해 결정된다.
- 하지만, [[Job]]의 그것은 설정에 따라 달라진다.
- **기본**
	- Step의 ExitStatus가 FAILED로 끝나면 Job의 ExitStatus와 BatchStatus는 모두 FAILED가 된다.
	- 이외에는 둘 다 COMPLETED가 된다.
- 위 케이스로 보통 충분하지만, 아닌 경우도 존재한다.