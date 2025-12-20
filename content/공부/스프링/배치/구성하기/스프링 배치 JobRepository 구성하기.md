- [공식 문서](https://docs.spring.io/spring-batch/reference/job/configuring-repository.html)
- [[스프링 배치 Job|JobExecution]], [[스프링 배치 Step|StepExecution]]과 같은 도메인 객체의 기본 CRUD 작업에 사용된다.
- 가장 간단한 구현체는 `ResourcelessJobRepository` 이며, 아무런 메타데이터를 저장하지 않는다.
	- 당연하게 [[스프링 배치 ExecutionContext|ExecutionContext]]를 지원하지도 않는다.
	- **기본적으로 `@EnableBatchProcessing` 또는 `DefaultBatchConfiguration`을 사용할 때 제공된다.**
- 데이터베이스 기반의 구현체는 JDBC 기반과 MongoDB 기반이 제공된다.
- JDBC 기반으로 구성하기 위해서는 `@EnableJdbcJobRepository`를 사용한다.
```java
@EnableJdbcJobRepository(
		dataSourceRef = "batchDataSource",
		transactionManagerRef = "batchTransactionManager",
		tablePrefix = "BATCH_",
		maxVarCharLength = 1000,
		isolationLevelForCreate = "SERIALIZABLE")
```
- 위 방법으로 구성할 수 없는 경우, `JdbcJobRepositoryFactoryBean`을 사용하거나, `SimpleJobRepository`를 사용해야 한다.