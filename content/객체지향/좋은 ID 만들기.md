- ID는 다음과 같은 요건을 고려하자.
    - 고유성 : ID는 각 객체를 정확하게 식별할 수 있어야 한다. 따라서 모든 객체는 고유한 ID를 가져야 한다.
    - 식별 가능성 : 굳이 저장소를 거치지 않더라도 값만으로 어느정도의 정보를 이해할 수 있으면 좋다.
    - 보안성 : ID는 예측하기 어려워야 한다. 순차적으로 증가하는 형태는 위험하다.
    - DB 성능 : ID는 삽입순으로 정렬되어야 한다. 그러지 않으면 INDEX 재정렬(?)이 계속 일어난다.

## 자연키

- 이메일, 주민등록번호 등 자연적으로 유일한 값을 사용한다.
- 고유하며, 식별가능하지만 DB 성능면에서 순서가 없기에 단점이 존재한다.
- 또한 비즈니스 요구사항이 변화하게 되면 그 변경이 DB까지 전파될 수 있다. (이메일 변경 불가 → 변경 가능)
- 따라서 자연키를 PK로 사용하는 것은 위험하다. PK로는 인조키를 사용하자.

## GeneratedValue

- DB에 의존하는 방식이다.
- `IDENTITY`
    - DB의 `AUTO_INCREMENT` 기능을 활용한다.
    - 삽입 후에 ID를 알 수 있기 때문에, 배치 삽입이 불가하다.
- `SEQUENCE`
    - DB의 `SEQUENCE` 기능을 활용한다.
    - 트랜잭션 커밋 전에도 ID를 알 수 있다. → 배치 삽입 가능
    - MySQL에서는 지원이 안된다… (또르륵)
- 결국 DB에 모두 의존한다.
- ID 자체가 이미 DB 의존적인 개념이라고 생각한다면 이런 방법을 사용하는 것도 좋을 것 같다.
- 하지만 ID가 도메인 엔티티 자체로 필수적인 개념이고, 그래서 DB에 의존하지 말아야 한다는 의견도 존재한다. → 이런 경우에는 UUID 계열을 고려하게 된다.
- 또한 예측할 수 있기 때문에 보안적으로 위험하다.

## 애플리케이션 수준에서 만들어보자.

- 토스의 ID 형태 : `test_ck_D5GePWvyJnrK0W0k6q8gLzN97Eoq`
    - test → 테스트 용도라는 것을 알 수 있다.
    - ck → 클라이언트키라는 것을 알 수 있다.
- 이 개념에 정렬가능성을 위하여 타임스탬프를 추가하면 좋을 것 같다.
- 최종 형태
    - [환경]_[엔티티명]_[타임스탬프&랜덤값]
    - 타임스탬프&랜덤값은 SnowFlake를 활용하면 생성할 수 있다.

```java
// snowflake id 생성용 의존성 추가
// implementation("com.github.f4b6a3:tsid-creator:5.2.6")

// 코드
public record DemoId(
    String value
) {
  private static final String VALUE_FORMAT = "%s_%s_%s";

  public static DemoId generate(String env, String entityName) {
    String snowflake = createSnowflakeId();
    return new DemoId(VALUE_FORMAT.formatted(env, entityName, snowflake));
  }

  private static String createSnowflakeId() {
    return TsidCreator.getTsid().toString();
  }
}

// 테스트
class DemoIdTest {

  @Test
  void 생성된_ID는_시간순으로_정렬된다() {
    List<DemoId> ids = new ArrayList<>();
    for (int i = 0; i < 10000; i++) {
      ids.add(DemoId.generate("test", "member"));
    }

    for (int i = 0; i < 9999; i++) {
      DemoId cur = ids.get(i);
      DemoId next = ids.get(i + 1);
      assertTrue(cur.value().compareTo(next.value()) < 0);
    }
  }

  @Test
  void 생성된_ID는_중복이_없다() {
    List<DemoId> ids = new ArrayList<>();
    for (int i = 0; i < 10000; i++) {
      ids.add(DemoId.generate("test", "member"));
    }

    Set<DemoId> set = new HashSet<>(ids);
    assertEquals(set.size(), ids.size());
  }
}
```