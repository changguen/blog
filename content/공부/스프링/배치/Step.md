> [[Job|Job]]을 구성하는 독립적인 단계

![[Pasted image 20251220145458.png]]
- 모든 [[Job]]은 하나 이상의 Step으로 구성된다.
- Step은 `JobRepository`가 필수적이며, [[TransactionManager]]는 선택적이다.
	- [[TransactionManager]]가 설정되지 않은 경우에는 `ResourcelessTransactionManager`가 사용되며, 트랜잭션이 작동하지 않는다.
- Step은 청크지향과 태스클릿으로 구분되며, 대부분 청크지향을 사용하고 태스트클릿은 매우 간단한 구현에만 사용한다.
- Step은 Reader/Writer로 구성되며 그 사이에 Processor를 선택적으로 넣는다.
## Tasklet 기반 Step
- 단일 트랜잭션으로 한번에 작업을 수행한다.
- 읽기-처리-쓰기 패턴에 상관없이 편하게 원하는 작업을 수행할 수 있다.
- 대용량 데이터 처리시 메모리 문제가 발생할 수 있다.
## Chunk 지향 Step
![[Pasted image 20251221230224.png]]
- 데이터를 청크 단위로 묶어서 트랜잭션을 보장한다.
- [[ItemReader]]로 데이터를 하나씩 읽어서 chunk 크기만큼 쌓고, 가공한 후 한번에 저장한다.
- 실패한 청크부터 재시작할 수 있다.
- 청크 단위로 메모리 사용을 일정하게 유지할 수 있다.
- 구성 요소
	- **[[ItemReader]]** : 데이터를 **하나씩** 읽는다.
	- **ItemProcessor** : 데이터를 가공하거나 변환하기 (선택적!)
	- **[[ItemWriter]]** : 가공된 데이터를 **청크 단위로** 한번에 특정 대상에 쓴다.
- 즉, 청크가 10이면 다음처럼 돈다.
  Reader : 10번
  Processcor : 10번
  Writer : 1번
## StepExecution
- `Step`을 실행하려는 단일 시도이다.
- 실제로 `Step`이 실행되기 전까지는 생성되지 않는다.
  예를 들어, 이전 `Step`이 실패하여 현재 `Step`이 실행도 안되었다면 `StepExecution`은 생성되지 않는다.
- 각 `Step`은 개별 `StepExecution`을 가진다.
- 다음 구성 요소를 가진다.
	- Status : 실행/시작/종료 상태
	- start/endTime
	- [[ExecutionContext|executionContext]] : 실행 간에 지속되어야 하는 데이터
	- read/writeCount : 읽은/기록된 항목의 수
	- commit/rollbackCount : 커밋된 트랜잭션의 수
	- rollbackCount : 트랜잭션이 롤백된 횟수
	- readSkipCount : 읽기 실패하여 스킵된 항목 수
	- processSkipCount : 프로세스가 실패하여 스킵된 항목 수
	- writeSkipCount : 쓰기가 실패하여 스킵된 항목 수
	- filterCount : 필터된 항목 수
## Step 재시도 설정
- 여기서 말하는 것은 [[스프링 배치 실패 동작]]에서 말하는 '자동 재시도'와 다르다.
  사람이 직접 실패한 Job을 재시도하는 것을 말하며, 해당 글의 자동 재시도는 스프링 배치에 의해 자동적으로 즉시 수행된다.
- startLimit : JobExecution이 실패하여 재시도 할 때, 해당 step을 몇번까지 재시도할 수 있는지 구성할 수 있다.
  즉, StepExecution의 최대 개수를 제한한다.
  기본값 : MAX
- allowStartIfComplete : 해당 Step이 실패한 것과 상관없이 재실행 하는 여부이다.
  즉, 이후 Step에서 실패하더라도 해당 Step은 같이 재실행 된다.
  ex) 임시 파일 삭제 step, 유효성 체크 step
- 아래 예시는 다음 설정이다.
	- 축구 Job은 플레이어로드 / 게임로드 / 플레이어요약 step으로 구성된다.
	- 플레이어로드 step은 실패시 무한 재시도 가능하며, 성공시 다시 실행되지 않는다.
	- 게임로드 step은 실패시 무한 재시도 가능하며, 성공하더라도 Job 재시도시 다시 실행된다.
	- 플레이어요약 step은 실패시 1번까지 재시도 가능하며(처음+재시도), 성공시 다시 실행되지 않는다.
```java
@Bean
public Job footballJob(JobRepository jobRepository, Step playerLoad, Step gameLoad, Step playerSummarization) {
	return new JobBuilder("footballJob", jobRepository)
				.start(playerLoad)
				.next(gameLoad)
				.next(playerSummarization)
				.build();
}

@Bean
public Step playerLoad(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("playerLoad", jobRepository)
			// ...
			.build();
}

@Bean
public Step gameLoad(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("gameLoad", jobRepository)
			.allowStartIfComplete(true)
			// ...
			.build();
}

@Bean
public Step playerSummarization(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("playerSummarization", jobRepository)
			.startLimit(2)
			// ...
			.build();
}
```