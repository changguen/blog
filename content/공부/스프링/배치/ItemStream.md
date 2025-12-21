- 데이터를 계속 읽기만 하다가 오류가 발생하면, 다음 시도에서 다시 처리해야 한다.
- 이런 문제를 해결하기 위하여 [[ExecutionContext]]를 통해서 상태를 저장하고, 이를 바탕으로 중간부터 재시도가 가능하다.
- `ItemStream`을 상속하도록 만들면 **`StepExecutionContext`** 를 받아볼 수 있다.
```java
public interface ItemStream {
	default void open(ExecutionContext executionContext) throws ItemStreamException {
	}

	default void update(ExecutionContext executionContext) throws ItemStreamException {
	}

	default void close() throws ItemStreamException {
	}
}
```
## 주의 : ItemStream은 스프링에 의해서 주입된다.
- 따라서 Reader 가 내부 필드로 가진 Reader에게 작업을 위임하는 등의 패턴에서는 `ExecutionContext`를 제대로 받아보지 못한다.
- 이 경우 스프링에게 '얘한테도 ExecutionContext 넣어줘' 라고 말해야 하는데, 이는 [[Step]] 구성 과정에서 수행한다.
```java
@Bean
public Step step1(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("step1", jobRepository)
				.<String, String>chunk(2).transactionManager(transactionManager)
				.reader(itemReader())
				.writer(compositeItemWriter())
				.stream(fileItemWriter1())
				.stream(fileItemWriter2())
				.build();
}
```