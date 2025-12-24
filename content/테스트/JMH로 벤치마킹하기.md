- 지구상 두 좌표의 거리를 측정하는 로직이 필수적으로 필요한데, 처음에는 이를 자바 코드로 구현했었다.
- 근데 더 찾아보니까 postgresql에서 이를 지원하는 것이었다.(postgis)
- 백엔드팀은 선택을 해야 했다.
- **기존 코드(자바로 구현)을 유지하는 선택**
    - 단점
        - 추후에 해당 코드를 읽기 매우 어려워질 것 같다. (벡터 내적, 하버사인 공식등의 수학적 지식이 필요하다.)
        - 성능이 postgis에 비해 낮을 것 같다. (얼만큼?)
    - 장점
        - 수정할 필요가 없으므로 다음 기능을 개발하면 된다.
- **postgis를 활용하는 선택**
    - 단점
        - DB 구현체가 postgresql로 고정되어야 한다.
        - postgresql을 팀 내에서 다뤄본 사람이 없었으므로, 따로 학습해야 한다.
        - 학습한 후에 적용하는 시간이 필요하다.
    - 장점
        - 직접 구현한 불안정한 코드보다 postgresql의 기능이 더 신뢰갈 수 있다.
        - 성능 자체는 기존보다 나아질 것 같다. (얼만큼?)
- 나머지는 이해가 가는데, 성능이 문제였다. postgis가 더 빠를 것 같긴 하지만 정확한 수치를 모르니 섣불리 도입하기 애매했다.
- 이 부분에서 벤치마킹 도구가 필요했고 여러 도구를 찾아보다 JMH를 도입했다.

## JMH란?

- JVM 기반 언어의 성능을 정확히 측정하기 위한 도구이다.
- `System.currentTimeMillis()` 같은 단순한 방법에 비해, JVM의 특성을 고려한 정교한 측정이 가능하다.
- 다음과 같은 것들을 설정할 수 있다.
    - 측정할 것: 초당처리량 / 평균 실행 시간 / 단일 실행 시간 …
    - 측정 단위: 나노초, 마이크로초, 밀리초, 초
    - JVM 프로세서 개수: JVM 최적화의 영향을 최소화 할 수 있음
    - 사전 실행 개수: JVM 최적화를 완료하도록 함
    - 측정 횟수

## JMH로 벤치마킹하는 코드

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Benchmark)
@Fork(2)
@Warmup(iterations = 3, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
public class CourseRepositoryBenchmark {

    private CourseRepository courseRepository;
    private TestService sut;
    private ConfigurableApplicationContext context;

    @Setup(Level.Trial)
    public void setUp() {
        // Spring Boot 애플리케이션 컨텍스트 시작
        SpringApplication app = new SpringApplication(PostgisDemoApplication.class);
        app.setWebApplicationType(WebApplicationType.NONE);
        context = app.run();

        sut = context.getBean(TestService.class);
        courseRepository = context.getBean(CourseRepository.class);

        // 테스트 데이터 준비
        setupTestData();
    }

    @TearDown(Level.Trial)
    public void tearDown() {
        if (context != null) {
            context.close();
        }
        courseRepository.deleteAll();
    }

    @Benchmark
    public void withNativeQuery() {
        sut.nativeQuery();
    }

    @Benchmark
    public void withJavaLogic() {
        sut.javaLogic();
    }

    private void setupTestData() {
        for (int i = 0; i < 100; i++) {
            Course testCourse = CourseCreater.createSquareCourse(
                    CourseCreater.createRandomCenterLat(),
                    CourseCreater.createRandomCenterLon(),
                    "Test Course " + i);
            courseRepository.save(testCourse);
        }
    }
}
```

## 고려사항

- 일단 두 방법 모두 최대한의 성능을 낸다는 가정하에 비교를 해야 했다.
- 따라서 lazy-loading, 인덱스 등의 최적화를 최대한 하고 비교했다.
- 또한 테스트데이터가 도메인과 얼마나 비슷하냐에 따라서 성능이 다를 것이라고 생각했다. → 최대한 실제와 비슷한 데이터를 삽입했다.
    - 서울 내에 100개 정도의 러닝 코스가 있다고 가정
    - 각 코스는 20개의 좌표로 구성되어 있다고 가정
    - 각 코스는 평균 길이가 3km 라고 가정

## 벤치마킹 결과

![[JMH로 벤치마킹하기_image.png]]

- 역시나 postgis가 더 빨랐지만, 엄청나게 차이난다…
- 100개 코스만에 0.1초 이상 차이나는 건데, 데이터가 더 많아진다면? 꽤나 고려해야 하는 부분이다.
- 물론 이것만으로 postgis를 바로 도입하지는 않을 것이고, 일단 과거 코드를 사용하다가 속도가 느려서 탈주하는 사용자가 많아지면 그 때 도입하면 될 것 같다. (피드백의 사이클을 짧게 가져가기)