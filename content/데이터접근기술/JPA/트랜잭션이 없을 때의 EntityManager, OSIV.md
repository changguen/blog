- 트랜잭션이 없어도 repository의 메서드는 EntityManager를 사용한다.
    - 정확히 말하면, repository 기본 구현체인 SimpleJpaRepository에는 각 메서드별로 트랜잭션이 걸려있다.
    - 따라서 이 엔티티 매니저가 각 repository 메서드 단위로 생성되었다가 삭제된다.
- **Repository의 모든 메서드는 EntityManager를 통한다!**

## OSIV가 true라면?

- 하지만 이렇게 잠깐 생성되어야 할 EntityManager가 OSIV를 만나면 달라진다.
- OSIV는 view 렌더링까지 계속 EntityManager를 살아있게 하므로, 여러 메서드가 동일한 엔티티매니저를 공유한다.
- 또한 지연로딩도 정상적으로 작동되어 버린다.