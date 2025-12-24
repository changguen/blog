> 반복 작업을 정형화하고 전략적으로 관리하기 위한 프레임워크, [링크](https://docs.spring.io/spring-batch/reference/repeat.html)

- `RepeatOperations` : 반복 작업의 추상화, 콜백을 넘겨서 돌린다.
  기본적인 구현체는 `RepeatTemplate`이다.
```java
public interface RepeatOperations {
    RepeatStatus iterate(RepeatCallback callback) throws RepeatException;
}
```
- `RepeatCallback` : 반복 작업 그 자체
- `RepeatContext` : 여러 콜백 호출 간에 유지되는 컨텍스트
- `RepeatStatus` : 반복 작업이 남아있는지 여부 (CONTINUABLE / FINSIHED)
```java
public interface RepeatCallback {  
   RepeatStatus doInIteration(RepeatContext context) throws Exception;  
}
```