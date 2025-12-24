- (이름 참 길다;;)
- 스프링부트 실행시 자동으로 `docker compose up`을 하고, 관련된 정보를 **스프링 설정정보에 추가**해주는 유용한 기능이다.
- 예를 들어, mysql을 여기에 설정하면, 로컬에서 스프링부트를 실행할 때 자동으로 도커도 띄워져서 로컬환경 테스트가 매우 쉬워진다. 또한 [[DataSource]] 관련 설정 정보가 자동으로 추가된다!

## 설정방법

1. `springboot docker compose support` 의존성 추가
    
    ```kotlin
    // build.gradle.kts
    
    dependencies {
        developmentOnly("org.springframework.boot:spring-boot-docker-compose")
    }
    ```
    
2. `compose.yaml` 을 프로젝트 루트에 작성
    
    ```yaml
    # compose.yaml
    
    services:
      mysql:
        image: 'mysql:latest'
        environment:
          - 'MYSQL_DATABASE=testdb'
          - 'MYSQL_PASSWORD=secret'
          - 'MYSQL_ROOT_PASSWORD=verysecret'
          - 'MYSQL_USER=spring'
        ports:
          - '3306:3306'
    ```
    
3. 라이프 사이클 설정
    
    ```yaml
    # application.yml
    
    spring:
      docker:
        compose:
          lifecycle-management: start_only
          # 시작만 같이 하고, 종료는 따로 하지 않는다. (기본값 : 실행과 종료를 같이)
          # 여기서 원한다면 compose.yaml 말고 다른 docker-compose 파일을 사용하는 설정도 존재한다.
    ```
    
4. 실행! 동적으로 다음 설정 정보가 추가된다. (볼 수는 없다.)
    
    ```yaml
    spring:
    	datasource:
    		url: jdbc:mysql://localhost:3306/testdb
    		username: spring
    		password: secret
    		driver-class-name: com.mysql.cj.jdbc.Driver
    ```
    

## 로컬에서만 실행되도록 제한하기

- `spring.docker.compose.enabled` 설정을 통해 해당 기능을 키고 끌 수 있다.

```yaml
# application.yml
# 기본적으로 비활성화 시켜놓는다.
spring:
  docker:
    compose:
	    enabled: false
	    
# application-deploy.yml
# 실제 DB의 설정 정보를 작성한다.
spring:
	datasource:
		url: jdbc:mysql://localhost:3306/testdb
		username: spring
		password: 1xn9u1xe90un10unx0u1e
		driver-class-name: com.mysql.cj.jdbc.Driver
		
# application-local.yml
# docker compose support를 통해 설정 정보를 구성한다.
spring:
  docker:
    compose:
	    enabled: true
	    
# application-test.yml
# 테스트용 DB등을 사용하여 설정 정보를 구성한다. (docker compose support를 그대로 사용해도 괜찮을 것 같다.)
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
```