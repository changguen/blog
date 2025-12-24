- Java 객체의 **데이터 유효성을 애노테이션으로 검증하는 표준 스펙**이다. (JSR-303)
- 데이터 유효성 검사하기
    - `jakarta.validation` 패키지의 `@NotNull` , `@Email` 등을 통하여 제약조건을 정의한다.
    - 이것만으로 검사되는 것은 아니며 `jakarta.validation` 패키지의 `@Valid` 애노테이션을 붙여야 된다.
    - `@NotNull` 등으로 제약조건을 걸고, `@Valid` 로 검사를 구현하는 느낌이다.
    - 이렇게 마킹하면 **런타임**에 Hibernate Validator가 실제 검사를 수행한다.
- 스프링에서 `@Valid` vs `@Validated`
    - `@Valid` 는 ArgumentResolver 수준에서 검사가 된다.
        - 중첩 객체 검증을 지원한다.
    - 스프링의 `@Validated` 는 AOP 수준에서 검사가 된다.
        - 직접 파라미터만을 검증한다.
	        - 검증 그룹을 지원한다. (나중에 공부!)
## 실용적인 컨트롤러와 서비스에서의 검증

- 컨트롤러
    - DTO : `@Valid` 를 파라미터에 붙이면 **ArgumentResolver** 수준에서 검사가 된다.
    - 직접 파라미터(PathVariable, RequestParam) : 별도의 ArgumentResolver에서 알아서 검사한다. 각 애노테이션의 옵션을 설정하자.
- 서비스
    - `@Valid` 를 파라미터에 붙여도 검사되지 않는다. ArgumentResolver를 타지 않기 때문이다. → `@Validated` 를 사용할 필요가 있다.
    - DTO :  `@Validated` 를 클래스에 사용하고, `@Valid` 를 통해 중첩 객체 검사를 한다.
    - 직접 파라미터 : `@Validated` 만으로 검사된다.
### Best Practice
```java
@RestController
public class MyController {

  @GetMapping("/withDirect/{name}")
  public void withDirect(@PathVariable("name") String name) {
    System.out.println("name = " + name);
  }
  
  public record DemoRequest(@NotNull String name) { }

  @GetMapping("/withDto")
  public void withDto(@Valid DemoRequest request) {
    System.out.println("request = " + request);
  }
}
```

```java
@Service
@Validated
public class MyService {

  public void withDirect(@NotNull String name) {
    System.out.println("name = " + name);
  }
  
  public record DemoDto(@NotNull String name) { }

  public void withDto(@Valid DemoDto dto) {
    System.out.println("name = " + dto.name());
  }
}
```

## 인터페이스를 사용하는 경우

- `@NotNull` 은 마킹, `@Valid` / `@Validated` 는 구현이다.
- 따라서 인터페이스에 `@NotNull` 등을 붙여서 시그니처를 정의하고, 실제 구현체에서 `@Valid` / `@Validated` 를 붙이면 된다.

### Best Practice

```java
public interface MyService {

  void runWithParam(@NotNull String name);

  record DemoDto(@NotNull String name) { }

  void runWithDto(DemoDto dto);
}
```

```java
@Service
@Validated
public class MyServiceImpl implements MyService {

  @Override
  public void withDirect(String name) {
    System.out.println("name = " + name);
  }

  @Override
  public void withDto(@Valid DemoDto dto) {
    System.out.println("name = " + dto.name());
  }
}
```

## 참고 : 스프링의 @NonNull

- **현재 JSpecify가 정식으로 채택되었다. 사용하면 안된다.**
- ~~자카르타의 `@NotNull` 등은 런타임에 실제 검증을 수행하는 목적이다.~~
- ~~이에 반해 스프링의 `@NonNull` 은 컴파일 타임 정적 분석용이다. (IDE나 분석도구에서 사용하기 위함)~~
    - ~~즉, 모든 객체 타입에 null이 들어갈 수 있다는 자바의 한계를 뛰어넘기 위한 노력이다.~~
    - ~~메서드 파라미터와 리턴값에 주요하게 사용한다.~~
- ~~정적 검사는 intelliJ IDEA, SpotBugs등을 통해 수행하면 된다.~~
    - ~~팁 : gradle 빌드가 SpotBugs의 검사에 종속적이게 설정하면 놓친 문제를 확인해볼 수 있다.~~
