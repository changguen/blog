- 메타가 개발한 API용 쿼리 언어
- 기존 REST API는 리소스 중심적이었다. `GET /user/123` : 123번 유저를 가져와라.
- 하지만 GraphQL은 쿼리 중심적이다. (진짜 JSON 상하차가 된 것인가? ㅋㅋ)ㅂ
- 기존 REST에서는 필요한 데이터에 비해 너무 많이 또는 적게 들어오는 문제가 있었다. (under/over fetching)
- 하지만 graphql에서는 정확히 원하는 데이터만 가져올 수 있다.
```text
query {
	user(id: "123") {
		name
		email
		posts {
			title 
		}
	}
}
```
- REST와 다르게 `/graphql` 단일 엔드포인트만 열어두면 된다.
- 스키마를 통해 API의 모든 타입과 관계를 명확히 정의해놓는다. (타입 안정성)
	- 타입 정의를 통해 클라이언트가 어떻게 요청할지 이해할 수 있다.
	- IDE나 프레임워크의 지원을 받을 수도 있다.
```text
type User {
	id: ID!
	name: String!
	email: String!
	posts: [Post!]!
}

type Post {
	id: ID!
	title: String!
	content: String!
	author: User!
}
```
- 대신에 graphql은 쿼리가 매번 다르기 때문에 캐싱하기 어렵다.
### 직접 실습해보기
`org.springframework.boot:spring-boot-starter-graphql` 의존성을 통해서 쉽게 개발할 수 있다.
#### 1. **DB/엔티티 만들기 (간소화)**
```java
@Entity
public class Member {  
	@Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;  
    private String name;  
    private String email;  
    @OneToMany(mappedBy = "member", cascade = CascadeType.ALL)  
    private List<Post> posts = new ArrayList<>();
    // ...
} 

@Entity  
public class Post {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String title;
    private String content;
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```
```java
public interface MemberRepository extends JpaRepository<Member, Long> {  
}

public interface PostRepository extends JpaRepository<Post, Long> {    
    List<Post> findByMemberId(Long memberId);  
}
```
#### 2. **스키마 정의하기**
```text
# resources/graphql/schema.graphqls

type Query {  
    members: [Member]  
    member(id: ID!): Member  
    posts: [Post]  
    post(id: ID!): Post  
    postsByMember(memberId: ID!): [Post]  
}  
  
type Mutation {  
    createMember(name: String!, email: String!): Member  
    createPost(title: String!, content: String!, memberId: ID!): Post  
}  
  
type Member {  
    id: ID!  
    name: String!  
    email: String!  
    posts: [Post]  
}  
  
type Post {  
    id: ID!  
    title: String!  
    content: String!  
    member: Member  
}
```
- `Post`, `Member`는 그래프 구조를 구성하고, `Query`, `Mutation`은 API 진입점을 의미한다.
- 예를 들어, `query members { id name posts { id } }` 에서
	- query members로 진입한다.
	- 중간 posts 부분에서 그래프를 타고 게시글에 대한 쿼리로 인식된다.
#### 3. **API 작성하기**
**Query**
```java
@Controller  
@RequiredArgsConstructor  
public class QueryResolver {  
  
    private final MemberRepository memberRepository;  
    private final PostRepository postRepository;  
  
    @QueryMapping  
    public List<Member> members() {  
        return memberRepository.findAll();  
    }  
  
    @QueryMapping  
    public Member member(@Argument Long id) {  
        return memberRepository.findById(id)  
                .orElseThrow(() -> new RuntimeException("Member not found"));  
    }  
  
    @QueryMapping  
    public List<Post> posts() {  
        return postRepository.findAll();  
    }  
  
    @QueryMapping  
    public Post post(@Argument Long id) {  
        return postRepository.findById(id)  
                .orElseThrow(() -> new RuntimeException("Post not found"));  
    }  
  
    @QueryMapping  
    public List<Post> postsByMember(@Argument Long memberId) {  
        return postRepository.findByMemberId(memberId);  
    }  
}
```
**Mutation**
```java
@Controller  
@RequiredArgsConstructor  
public class MutationResolver {  
  
    private final MemberRepository memberRepository;  
    private final PostRepository postRepository;  
  
    @MutationMapping  
    public Member createMember(@Argument String name, @Argument String email) {  
        Member member = new Member(name, email);  
        return memberRepository.save(member);  
    }  
  
    @MutationMapping  
    public Post createPost(@Argument String title, @Argument String content, @Argument Long memberId) {  
        Member member = memberRepository.findById(memberId)  
                .orElseThrow(() -> new RuntimeException("Member not found"));  
  
        Post post = new Post(title, content, member);  
  
        return postRepository.save(post);  
    }  
}
```
스키마의 API 진입점을 작성하게 된다.
#### 4. 사용해보기!
```text
### 모든 회원의 id, name, email 조회
POST localhost:8080/graphql  
Content-Type: application/json
  
{  
  "query": "query { members { id name email } }"
}

---

### 특정 회원의 id, name, email과 그 회원의 모든 게시글 조회
POST {{baseUrl}}/graphql  
Content-Type: {{contentType}}  
  
{  
  "query": "query { member(id: 1) { id name email posts { id title content } } }"
}
```
### 실습 후 생각
- graphql은 서버보단 클라이언트 지향적인 방법인 것 같다.
	- 코드를 짜는 것은 힘들었다...
	- 하지만 사용할 때는 매우 직관적이었다.
- graphql은 rest의 대체제가 아닌 것 같다.
	- 쿼리에는 확실히 매우 강력하다. 하지만 mutation 쪽은 잘 모르겠다. rest api와 사실 동일한 것 같다.
	- 또한 집계나 복잡한 로직이 들어가는 쿼리의 경우에는 rest api가 개발하기 더 편할 것 같다. (그래프를 잘 활용할 수 없으므로!)
	- 또한 여러 방식으로 쿼리할 필요가 없는 경우(depth가 매우 얇다던가)에는 rest api가 오히려 직관적일 수 있는 것 같다.
	- **따라서 필요한 경우 혼용해서 사용하게 될 것 같다.**
