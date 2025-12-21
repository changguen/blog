> [[Job|Job]]을 구성하는 독립적인 단계

![[Pasted image 20251220145458.png]]
- 모든 [[Job]]은 하나 이상의 Step으로 구성된다.
- **StepExecution**
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
## Tasklet 기반 Step
- 단일 트랜잭션으로 한번에 작업을 수행한다.
- 읽기-처리-쓰기 패턴에 상관없이 편하게 원하는 작업을 수행할 수 있다.
- 대용량 데이터 처리시 메모리 문제가 발생할 수 있다.
## Chunk 지향 Step
- 데이터를 청크 단위로 묶어서 트랜잭션을 보장한다.
- [[ItemReader]]로 데이터를 하나씩 읽어서 chunk 크기만큼 쌓고, 가공한 후 한번에 저장한다.
- 실패한 청크부터 재시작할 수 있다.
- 청크 단위로 메모리 사용을 일정하게 유지할 수 있다.
- 구성 요소
	- **[[ItemReader]]** : 데이터를 하나씩 읽는다.
	- **ItemProcessor** : 데이터를 가공하거나 변환하기 (선택적!)
	- **ItemWriter**
		- 가공된 데이터를 **청크 단위로** 한번에 특정 대상에 쓴다.
		- 다음 항목에는 접근하지 못하며, 현재 전달된 항목만에 집중한다.