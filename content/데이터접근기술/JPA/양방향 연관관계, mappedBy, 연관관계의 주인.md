> 자바에서 두 객체가 서로 참조하게 하려면?
→ 양 객체에 서로의 참조를 들고 있도록 한다.
테이블에서 두 객체가 서로 참조하게 하려면?
→ 한 테이블에 FK를 둔다.
> 

위 내용이 mappedBy, 연관관계의 주인 개념의 시작이다.

```java
@Entity
public class Book {

    @ManyToOne
    private Publisher publisher;
}

@Entity
public class Publisher {

    @OneToMany
    private Set<Book> books = new HashSet<>();
}
```

- 위 코드만으로 자바에서는 양방향 연관관계가 성립하지만, 테이블에서는 다르다.
- **JPA는 위 두 관계가 각각 의미를 가진 것으로 해석한다.**
- 따라서 두 연관관계를 모두 표현하려고 노력한다.
    - `N : 1` 인 Book 쪽에서는 Book 테이블에 칼럼을 하나 추가하여 `pulisher_id` 칼럼을 만든다.
    - `1 : N` 인 Publisher 쪽에서는 새로운 **연결 테이블**을 만들어, 관계를 표현한다. (칼럼 : `publisher_id`, `book_id`)
- 또한 따라서 Book에 Publisher를 세팅해서 영속화한다고 해도 Publisher의 Set<Book>에는 해당 정보가 없다.
    - 애초에 다른 관계인데, 왜 넣는가?
    - 한 사람이 내 친구라고 해서, 그 사람이 내 애인인 것은 아니다.
- 이 문제를 어떻게 해결할까?

```java
@Entity
public class Book {

    @ManyToOne
    private Publisher publisher;
}

@Entity
public class Publisher {

    @OneToMany(mappedBy = "publisher")
    private Set<Book> books = new HashSet<>();
}
```

- 위와 같이 표기하면, 두 참조가 동일한 연관관계를 나타내기 위함이란 것을 표현할 수 있다.
- 이러면 Book 테이블에 `publisher_id` 칼럼을 추가하는 것으로 관계가 표현된다. (테이블을 굳이 만들 필요 없으므로!)