> 가공된 데이터를 **청크 단위로** 한번에 특정 대상에 쓴다.

```java
public interface ItemWriter<T> {  
  
    void write(Chunk<? extends T> chunk) throws Exception;
}
```
- 청크를 모두 쓰도록 구현하면 된다.
- 다음 항목에는 접근하지 못하며, 현재 전달된 항목만에 집중한다.
- [[ItemStream]]을 통해서 [[ExecutionContext]]에 접근할 수 있다.
- 커스텀을 밑바닥 부터 구현해야 하는 경우 다음 링크를 참고하자. [링크](https://docs.spring.io/spring-batch/reference/readers-and-writers/custom.html)
## 제공되는 구현체
- DB 기반
	- `JdbcBatchItemWriter`
	- `JpaItemWriter`
	- `RepositoryItemWriter`
- 기타 기반
	- `FlatFileItemWriter`
	- `JsonFileItemWriter`
	- `KafkaItemWriter`
	- `AmqpItemWriter`
- 이외에도 더 많다. 필요한 경우 찾아보면 될 것 같다.
  [링크](https://docs.spring.io/spring-batch/reference/appendix.html#listOfReadersAndWriters)