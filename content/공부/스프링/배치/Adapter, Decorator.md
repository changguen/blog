- [[ItemReader]], [[ItemWriter]]를 구현할 때, **기존 서비스의 메서드를 활용해야 하는 경우** Adapter를 사용할 수 있다.
```java
@Bean
public ItemReaderAdapter itemReader() {
	ItemReaderAdapter reader = new ItemReaderAdapter();

	reader.setTargetObject(fooService());
	reader.setTargetMethod("generateFoo");

	return reader;
}

@Bean
public ItemWriterAdapter itemWriter() {
	ItemWriterAdapter writer = new ItemWriterAdapter();

	writer.setTargetObject(fooService());
	writer.setTargetMethod("processFoo");

	return writer;
}
```
- 또한 이미 구현된 [[ItemReader]], [[ItemWriter]]에 추가 기능을 넣고 싶은 경우 Decorator를 사용할 수 있다.
  ex) `synchronized`, 다음 아이템 피킹 등
- 여러 Decorator가 있어서, 다음 링크를 참고하자. [링크](https://docs.spring.io/spring-batch/reference/readers-and-writers/item-reader-writer-implementations.html)