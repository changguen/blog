- [[ItemReader]], [[ItemWriter]]를 구현할 때, 기존 서비스의 메서드를 활용해야 하는 경우 Adapter를 사용할 수 있다.
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
