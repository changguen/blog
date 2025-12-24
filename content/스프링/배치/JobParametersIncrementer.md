> 다음 실행을 위한 새로운 `JobParameters`를 생성한다.

- [[Job]]은 이름+`JobParameter`로 `JobInstace`가 구분된다.
	- 한 `JobInstace`가 성공했다면, 다시 실행시킬 수 없다.
	- **이 제약을 풀기 위해 주로 사용한다.**
- 또는 `JobParameters`를 자동 생성하는 개념으로도 사용한다.
```java
public interface JobParametersIncrementer {
	JobParameters getNext(JobParameters parameters);
}
```
- 기본적인 구현체는 `RunIdIncrementer`이며, 파라미터에 run.id 라는 키를 추가하여 매 실행마다 1씩 증가시킨다.
  -> 매번 `JobParameters`가 고유해지기 때문에 중복 실행이 가능해진다.
- Job 구성시에 추가하면 된다.
```java
public Job importUserJob(JobRepository jobRepository, Step step1) {
    return new JobBuilder("importUserJob", jobRepository)
        .incrementer(new RunIdIncrementer())
        .start(step1)
        .build();
}
```
