```java
public interface ItemProcessor<I, O> {
    O process(I item) throws Exception;
}
```
- [[ItemReader]]와 [[ItemWriter]] 사이에서 추가적인 비즈니스 로직을 처리한다.
- **가공하기** : 그냥 변환해서 응답하면 된다.
- **필터링하기** : null을 응답하면 된다.
- **검증하기** : 예외를 던지면 된다.