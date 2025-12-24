- 정적 의존 관계 : `Repository` ← `CrudRepository` ← `PagingAndSortingRepository` ← `JpaRepository`
- 동적 의존 관계
    - 내가 선언한 인터페이스를 구현한 **프록시 객체**가 동적으로 만들어진다.
    - 기본 메서드
        - `SimpleJpaRepository` 가 `JpaRepository` 를 구현해놓았다.
        - 따라서 기본적인 `JpaRepository` 의 메서드는 `SimpleJpaRepository` 에게 위임한다.
    - 커스텀 메서드
        - 메서드명 또는 `@Query` 내부 값을 통해 `SimpleJpaQuery` 를 생성해놓는다.
        - 런타임에는 해당 쿼리를 실행한다.

```java
public interface MyRepository extends JpaRepository<Member, Long> {
	List<Member> findByName(String name);
	
	@Query("...")
	List<Member> findByName2(String name);
}

public class ProxyRepository extends MyRepository {

	private final SimpleRepository simpleRepository;
	private final Map<Method, SimpleJpaQuery> queries;
	private final EntityManager entityManager;
	
	// 기본 메서드
	// SimpleJpaRepository에 위임한다.
	public Member save(Member member) {
		return simpleRepository.save(member);
	}
	
	// 내가 정의한 메서드 : 메서드 이름 추론
	// 리플렉션으로 이름 분석하여 미리 SimpleJpaQuery로 저장해놓는다.
	// 런타임에는 SimpleJpaQuery를 실행한다.
	public List<Member> findByName(String name) {
		Method method = ...; // 현재 메서드
		RepositoryQuery query = queries.get(method);
		return query.execute(new Object[]{name});

	}
	
	// 내가 정의한 메서드 : @Query 사용
	// @Query의 내용을 바탕으로 EntityManager를 통해 쿼리를 생성하여 저장해놓는다.
	// 런타임에는 SimpleJpaQuery를 실행한다.
	public List<Member> findByName2(String name) {
		Method method = ...; // 현재 메서드
		RepositoryQuery query = queries.get(method);
		return query.execute(new Object[]{name});
	}	
}
```