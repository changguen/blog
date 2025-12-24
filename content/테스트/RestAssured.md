> 학습 테스트에 포함되어 있던 코드인데, 미션할 때에는 중요하게 생각하지 않았는데, 막상 모르는 것이라 공부
ChatGPT를 통해 공부
> 
- `RestAssured`는 자바 코드로 API 요청을 작성할 수 있게 도와주는 라이브러리
- 또한 API 응답 역시 자바 코드로 검증할 수 있도록 한다.

```java
RestAssured.given()
    .contentType("application/json")
    .body("{ \"name\": \"changgeun\" }")
.when()
    .post("/users")
.then()
    .statusCode(201)
    .body("id", notNullValue());
```

## MockMvc VS RestAssured

- `MockMvc` 는 다른 부분을 모킹하여 해당 컨트롤러 부분만을 단위테스트하는데 특화 (테스트용 Request, Response 사용)
- `RestAssured` 는 실제 API 요청을 통해 전체적인 통합테스트에 특화

## **RestAssuredMockMvc**

- RestAssured의 DSL 문법이 마음에 드는데, 실제 테스트는 MockMvc 처럼 하고 싶을 때 선택할 수 있는 방법이다.
- MockMvc를 그저 RestAssured의 문법으로 감싸서 제공한다.

```java
RestAssuredMockMvc.given()
    .standaloneSetup(new UserController())
    .contentType("application/json")
    .body("{ \"name\": \"changgeun\" }")
.when()
    .post("/users")
.then()
    .statusCode(201)
    .body("id", notNullValue());
```