> spring-learning-test 에서 멀티 모듈이 사용되었길래 어떤식으로 구성될 수 있는지 궁금해서 학습
해당 학습 테스트 내용처럼 만들어보며 실습
> 

## 디렉토리 구조

```

├── gradle
├── gradlew
├── gradlew.bat
├── build.gradle
├── domain
│   ├── src
│   │   ├── main
│   │   └── test
│   └── build.gradle
├── web
│   ├── src
│   │   ├── main
│   │   └── test
│   ├── build.gradle
├── infra
│   ├── src
│   │   ├── main
│   │   └── test
│   └── build.gradle
└── settings.gradle
```

## 루트 모듈

### settings.gradle

```groovy
rootProject.name = 'multi-module-demo'

include 'web'
include 'domain'
include 'infra'
```

- 루트 프로젝트명과 어떤 서브 모듈들을 가지는지 명시한다.

### build.gradle

```groovy
allprojects {
    apply plugin: 'java'

    group = 'dompoo'
    version = '1.0-SNAPSHOT'

    repositories {
        mavenCentral()
    }
}

subprojects {
    dependencies {
        testImplementation platform('org.junit:junit-bom:5.10.0')
    }

    test {
        useJUnitPlatform()
    }
}

```

- 기존에 봐왔던 build.gradle과 많이 다르다.
- `allprojects` 의 요소들은 루트 모듈 포함, `subprojects` 의 요소는 미포함이다.
- 현재 설정은 다음과 같다.
    - 모든 모듈은 java 플러그인을 사용한다.
    - 모든 모듈은 group, version, repositories를 해당 값처럼 갖는다.
    - 하위 모듈들은 다음과 같은 요소들을 갖는다.

## 하위 모듈

### build.gradle

```groovy
plugins {
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'io.rest-assured:rest-assured:5.3.1'
}
```

- 루트 모듈에 이미 자바 관련 의존성을 추가했으므로 필요한 의존성과 플러그인만 작성하면 된다.