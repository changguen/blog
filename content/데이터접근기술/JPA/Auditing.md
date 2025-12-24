- **감사**라는 뜻이다. 뭔가 변경을 감지한다는 것!

### Spring Data JPA Audting

- **생성 시간, 수정 시간, 작성자, 수정자** 같은 정보를 기록할 수 있다.
- `@EnableJpaAuditing` 를 통해 활성화할 수 있다.
- 기능
    - `@CreateDate` : 처음 저장될 때 자동 저장
    - `@LastModifiedDate` : 수정될 때마다 갱신
    - `@CreatedBy` : 처음 저장될 때 작성자 저장
    - `@LastModifiedBy` : 수정될 때마다 수정자 갱신

### Hibernate Envers

- 엔티티의 **무엇을, 누가, 언제** 바꿨는지 기록할 수 있다.
- `~_AUD` 테이블을 통해 해당 기록을 저장한다.