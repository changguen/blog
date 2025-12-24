- 기본적으로 붙어있지 않는 옵션이다.
- 상대 엔티티에게 CRUD 연산을 전파하는 기능이다.
- 즉, cascade 옵션은 상대 엔티티를 관리하도록 도와주는 기능이다.
- [[양방향 연관관계, mappedBy, 연관관계의 주인|양방향 연관관계]]에서 특히 유용하며, [[orphanRemoval 옵션]]과 함께 자주 사용된다.
- 예를 들어, User가 List<Reservation>을 가지고 있을 때
    - User가 저장되면 Reservation도 같이 저장하고 싶을 수 있다.
    - 하지만, cascade가 없으면 Reservation 쪽은 따로 저장해야 한다.

```java
// cascade 안 붙인 경우
Team team = new Team();
Member member = new Member();
member.setTeam(team);

memberRepository.save(member);
// Team은 저장되지 않았다.
// 비영속 객체를 참조하고 있는데, 저장하려 한다고 예외가 터진다.

// cascade 붙인 경우
Team team = new Team();
Member member = new Member();
member.setTeam(team);

memberRepository.save(member);
// Member -> Team 으로 영속성 전파
// Team도 같이 insert 된다.
```

- 애그리게이트 루트 쪽에 설정하여 자식 엔티티들도 같이 저장될 수 있도록 하자!