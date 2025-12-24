- `Flow`는 [[Job]]을 구성하는 흐름 중 하나이며, [[Step 흐름 제어]]에서 나오는 모든 구성은 사실 `Flow`였다.
- 한 `Flow`는 선형적으로 작동한다.
- **여러 `Flow`는 병렬적으로 작동할 수 있다.**
```java
@Bean
public Flow flow1(Step step1, Step step2) {
	return new FlowBuilder<SimpleFlow>("flow1")
			.start(step1)
			.next(step2)
			.build();
}

@Bean
public Flow flow2(Step step3) {
	return new FlowBuilder<SimpleFlow>("flow2")
			.start(step3)
			.build();
}

@Bean
public Job job(JobRepository jobRepository, Flow flow1, Flow flow2, Step step4) {
	return new JobBuilder("job", jobRepository)
				.start(flow1)
				.split(new SimpleAsyncTaskExecutor())
				.add(flow2)
				.next(step4)
				.end()
				.build();
}
```
- 위 예시에서 flow1과 flow2는 병렬적으로 실행되며, 두개가 모두 성공한 경우에 step4가 실행된다.
- 더 정확하게 이해하면, step4도 하나의 flow를 구성하므로(명시적으로 드러나지는 않는다.) 다음의 구성을 가지는 것이다.
![[Pasted image 20251222102540.png]]
- 참고 : Flow를 굳이 병렬 처리를 위해서 사용하지 않아도 되는데, 말 그대로 '선형 흐름' 그 자체이기 때문이다.
  그냥 어떠한 중요한 흐름을 더 명시적으로 드러내고, 캡슐화하기 위해서 사용할 수도 있다.