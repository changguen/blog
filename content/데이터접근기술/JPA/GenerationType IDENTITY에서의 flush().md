- DB의 IDENTITY 칼럼을 통해 ID를 생성한다.
- 즉, DB에 저장해봐야 ID를 알 수 있다.
- 따라서 IDENTITY 전략을 쓸 경우, `em.persist()` 를 수행하자마자 insert 쿼리가 나간다.
- `flush()` 된다고 표현하는데, 그러면 다른 쿼리도 나갈 것 같다. 정확히는 insert 쿼리만 flush() 되는 것이다.

```java
@Transactional
public void doSomething() throws InterruptedException {
    Customer customer = new Customer("test", "test");
    customerRepository.save(customer);
    em.flush();
    Thread.sleep(3000);
    System.out.println("!테스트 시작!");

    System.out.println("!이름 수정!");
    customer.changeFirstName("newFirstName");
    System.out.println("!이름 수정 종료!");
    Thread.sleep(3000);

    System.out.println("!새로운 데이터 저장 -> flush 될까?!");
    System.out.println("!만약 flush 된다면 위 이름 수정 쿼리도 날라가야함!");
    customerRepository.save(new Customer("test2", "test2"));
    System.out.println("!새로운 데이터 저장 종료!");

    Thread.sleep(3000);
    System.out.println("!트랜잭션 종료!");
}
```

- 다만 insert 쿼리만 나가고, 다른 쿼리들은 정상적으로 더티체킹/쓰기지연 된다.
- 따라서 위 예시에서, update 쿼리는 트랜잭션 종료시 나가게 된다.