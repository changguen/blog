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