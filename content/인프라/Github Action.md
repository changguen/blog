# Runner

- github action을 사용하면 어떠한 컴퓨터(runner)를 사용하게 된다.
- `github hosted runner`
    - 깃허브의 컴퓨터를 빌린다. (설치 필요 X)
    - `runs-on: ubuntu-latest` 같이 설정하여 원하는 OS의 컴퓨터를 빌려서 사용한다.
    - 큰 컴퓨터와 작은 컴퓨터가 있는데, 작은 컴퓨터를 기본으로 사용하게 된다.
- `self hosted runner`
    - 내가 지정한 컴퓨터를 사용한다. (설치 필요)
    - `runs-on: self-hosted` 같이 설정하여 사용한다.
    - 정확히 어떤 컴퓨터를 사용하는지는 레이블을 통해 특정한다.
    - `runs-on: [self-hosted, dev-server-runner]` 처럼 설정하면 두 레이블이 모두 존재하는 runner를 사용한다. (개발 서버에서 실행됨)
- `self hosted runner` 연결 과정 (Long Polling)
    1. runner가 실행되고 준비 완료되면 GitHub 서버에 연결을 요청한다.
    2. GitHub 서버는 runner에게 할당할 작업이 생겼을 때까지 연결을 열어둔 채로 대기한다.
    3. 작업이 생기면 GitHub 서버가 기다리던 연결을 통해 작업을 전달한다.
    4. 작업이 일정 시간 없으면 GitHub는 작업이 없다고 알리고 연결을 종료한다. runner는 그 즉시 다시 연결을 요청하며 이것이 반복된다.
- 왜 Long Polling 인가?
    - 비용: runner가 연결을 열어두고 대기하는 대부분의 시간동안 Idle 상태에 놓여있기 때문에 리소스 사용량은 극히 작다.
    - 보안: push한다는 것은 GitHub 서버가 runner 서버로 직접 접근할 수 있어야 한다는 것이다.
        - 이는 방화벽을 열고 특정 포트를 열어두어야 한다는 뜻이며, 보안성이 안좋다.
        - 이에 비해 Long Polling 방식은 runner 서버에서 GitHub 서버로 연결을 먼저 만드는 것이기 때문에 이럴 필요가 없다.

# Custom Action

- 컴퓨터도 빌렸으니 이제 작업을 수행하면 된다.
- 하지만 처음부터 끝가지 다 쉘 스크립트를 작성하는 일은 쉽지 않다.
- 이런 경우 다른 유저나 깃허브가 미리 구현해둔 `Custom Action`을 가져다 쓸 수 있다.
    - 자바 설치하기
    - 테스트 리포트 만들기
    - 코드 체크아웃 하기
    - ...

# Caching

- `github hosted runner`의 경우 매번 새로운 컴퓨터를 사용하게 된다.
    - 따라서 설치한 java나 라이브러리등은 모두 날라가게 된다.
    - 이는 보안성을 위해서다. 이전 컴퓨터의 파일이 남아있게 되면 보안적으로 위험할 수 있다.
- 하지만 이러면 성능이 너무 안좋으므로 필요한 경우 캐시 기능을 사용해볼 수 있다.

```
- name: Cache Gradle packages
    uses: actions/cache@v4
    with:
      path: ~/.gradle/caches # ~/.gradle/caches 디렉토리를 캐싱 대상으로 지정
      key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle.kts') }} # build.gradle.kts 파일이 변경될 때만 캐시를 새로 생성
      restore-keys: |
        ${{ runner.os }}-gradle-

```