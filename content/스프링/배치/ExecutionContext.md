> Step/Job 단위로 관리되는 실행 컨텍스트

- [[Step|StepExecution]] / [[Job|JobExecution]] 범위에 속하는 지속성 상태를 저장할 수 있다.
  `Step/JobExecution.getExecutionContext()` 를 통해 가져온다.
- 재시작을 용이하게 만든다.
  [[ItemReader]] 등에서 데이터를 읽고 커밋을 계속한다.
  -> 중간에 오류가 발생해도 중간부터 시작할 수 있다.
- 작업이 재시작될 때, ExecutionContext 값은 데이터베이스에서 재구성된다.
  -> ExecutionContext를 주입받아 사용하면 된다.
- **값을 넣을 때 덮어쓰지 않도록 주의해야 한다.**
