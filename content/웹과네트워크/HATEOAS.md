- Hypermedia As The Engine Of Application State
- [[REST]]의 최고 성숙도 단계이다. (ㄷㄷ;)
- API 응답에 **다음 가능한 행동들의 링크를 포함**시켜 클라이언트의 하드코딩 없이 API를 탐색할 수 있도록 하는 것이다.
- 클라이언트는 초기 URI만 알면, 그 후로는 링크를 통해 동적으로 다음 행동을 취할 수 있다.
```json
{
  "id": 1,
  "name": "홍길동",
  "email": "hong@example.com",
  "_links": {
    "self": {
      "href": "http://api.example.com/users/1"
    },
    "orders": {
      "href": "http://api.example.com/users/1/orders"
    },
    "update": {
      "href": "http://api.example.com/users/1",
      "method": "PUT"
    },
    "delete": {
      "href": "http://api.example.com/users/1",
      "method": "DELETE"
    }
  }
}
```
- `org.springframework.boot:spring-boot-starter-hateoas` 의존성을 통해 쉽게 구현할 수 있다.
- **장점**
	- 서버측 URI 변경에도 클라이언트 코드 수정이 불필요하다.
	- 개발자가 API 구조를 더 쉽게 이해할 수 있다.
- **단점**
	- 응답 크기 증가
	- 복잡성 증가
	- 문서화 도구와의 궁합이 안좋음 (동적 URI를 캐치하기 어려움)
- 실용적으로 생각하면 다음과 같다.
	- 이론적으로는 우아하다.
	- 하지만 서버 개발 복잡성에 비하여 클라이언트 개발자가 얻는 이점이 적고, 성능적 오버헤드가 크기 때문에 채택하기 어렵다.
	- 유명 기업들도 `Github API`의 일부 정도를 제외하고는 찾아보기 어렵다.