- 리스너를 통해 특정 상황에 원하는 코드를 실행시킬 수 있다.
- `JobExecutionListener`와 `StepListener`가 있다.
- `JobExecutionListener`는 직접 구현해야 한다.
  [[Job|Job]]의 시작과 끝에 트리거된다.
```java
public interface JobExecutionListener {

    void beforeJob(JobExecution jobExecution);

    void afterJob(JobExecution jobExecution);
}
```
- `StepListener`는 하위 클래스들을 사용하여 구현한다. (마커 인터페이스)
	- `StepExecutionListener` : 스텝의 시작과 끝
	- `ChunkListener` : 청크의 시작과 끝, 예외 발생
	- `ItemReadListener` : 아이템을 읽기 전과 후, 예외 발생
	- `ItemWriteListener` : 아이템을 쓰기 전과 후, 예외 발생
	- ...
- 이것들을 구현하여 [[Job|Job]]과 [[Step|Step]] 구성시에 사용하면 된다. ([[스프링 배치 Job 구성하기]])