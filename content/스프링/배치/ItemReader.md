> 데이터소스로부터 **하나씩** 데이터를 읽는다.

```java
public interface ItemReader<T> {

    T read() throws Exception;
}
```

- 제공할 수 있는 항목을 모두 소진하면 null을 반환한다.
- 스프링이 여러 데이터소스에 맞는 빌더와 구현체를 제공한다.
  -> 보통 이것들을 가져다 쓰며, 필요한 경우 따로 정의해서 사용한다.
  -> 이동욱님의 [querydsl 기반 구현](https://github.com/jojoldu/spring-batch-querydsl) 도 유명하다.
- [[ItemStream]]을 통해서 [[ExecutionContext]]에 접근할 수 있다.
- 커스텀을 밑바닥 부터 구현해야 하는 경우 다음 링크를 참고하자. [링크](https://docs.spring.io/spring-batch/reference/readers-and-writers/custom.html)
## 제공되는 구현체
- DB 소스
	- 페이지 방식
		- `JdbcPagingItemReader`
		- `JpaPagingItemReader`
	- 커서 방식
		- `JdbcCursorItemReader`
		- `JpaCursorItemReader`
	- `RepositoryItemReader`
- 파일 소스
	- `FlatFileItemReader`
	- `JsonItemReader`
	- `StaxEventItemReader`
- 이외에도 더 많다. 필요한 경우 찾아보면 될 것 같다.
  [링크](https://docs.spring.io/spring-batch/reference/appendix.html#listOfReadersAndWriters)
## 참고 : Paging vs Cursor
- **커서 방식** : 한번에 조회해놓고 애플리케이션에서 조금씩 가져오기
	- (100 조회 -> 10 읽기 -> 10 읽기 -> 10 읽기 -> ...)
	- 따라서 청크 단위로 긴 트랜잭션을 갖는다.
	- 락문제/타임아웃에 위험하다.
	- 스냅샷 격리를 통해 데이터 일관성이 지켜진다.
	- 커서 위치를 저장하지 못하기 때문에 재시작이 어렵다. 또한 멀티스레드가 불가능하다.
- **페이징 방식** : 각각 OFFSET을 통해서 여러번 조회하고 그대로 가져오기
	- (10 조회 -> 10 읽기 -> 10 조회 -> 10 읽기 -> ...)
	- 따라서 페이지 단위로 짧은 트랜잭션
	- 락문제/타임아웃에서 비교적 안전하다.
	- 데이터 일관성 불일치 가능하다.
	- 페이지 번호로 재시작하기 쉽다. 또한 멀티스레딩이 가능하다.