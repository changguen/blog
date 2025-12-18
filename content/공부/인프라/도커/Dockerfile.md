> [[도커 이미지]]를 생성하기 위한 레시피다.

- Dockerfile에 적힌 명령어를 위에서부터 순서대로 실행하여 [[도커 이미지]]를 생성한다.
- 다음과 같은 순서(명령어)들이 존재한다.
	- `FROM` : 어떤 이미지를 바탕으로 시작할지 결정한다.
	  스프링 애플리케이션의 경우 자바나 gradle이 설치된 환경을 기준으로 시작한다.
	- `WORKDIR` : 명령어가 실행될 기본 디렉토리를 설정한다. (본격적 명령어 시작전에 cd한다)
	- `COPY` : 개발자 머신에 있는 파일이나 폴더를 이미지 내부로 복사한다.
	- `RUN` : 명령어를 실행한다.
	- `ARG` : 빌드를 할 때만 유효한 환경 변수이다.
	- `ENV` : 컨테이너 실행시에도 유지되는 환경 변수이다.
	- `EXPOSE` : 컨테이너 실행시 사용할 포트를 **문서화**한다. (실제 실행시 따로 포트를 열어야 된다.)
	  없다면, 이미지를 사용하는 입장에서 무슨 포트인지 확인하기 어렵다.
	- `CMD` : 컨테이너로 실행하는 순간 수행될 기본 명령어다.
	- `ENTRYPOINT` : 컨테이너로 실행하는 순간 수행될 prefix 명령어다.
- [[멀티 스테이지 빌드]]를 통해 이미지 크기를 줄일 수 있다.
- **각 명령어마다 레이어다.** 따라서 바뀌지 않는 부분을 위에 올리면 캐싱이 되어, 빌드 속도가 향상된다. ([[라이브러리 캐싱 레이어 분리]])
- 대부분의 도커 이미지는 root 사용자로 실행된다. 따라서 별도 사용자를 만들어 실행하는 것이 표준이다.
```dockerfile
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
```
- 참조 : `CMD` vs `ENTRYPOINT`
	- CMD는 해당 이미지를 실행할 때 기본적으로 사용할 명령어다.
	  실행시 다른 명령어를 주면 해당 명령어는 대체된다.
	  ex) 된장 찌개, 실행시 김치 찌개 -> 김치 찌개
	- ENTRYPOINT는 해당 이미지를 실행할 때 무조건적으로 사용할 명령어다.
	  실행시 다른 명령어를 주면 해당 명령어가 뒤에 붙는다.
	  ex) 김치, 실행시 찌개 -> 김치 찌개
- 참조 : `ARG` vs `ENV`
	- ARG는 빌드시 필요한 인자다.
	- ENV는 실행시 필요한 인자다.
	- 다음과 같이 설정하면 '빌드 시에 결정한 dev 프로필로 실행 시에도 작동'한다. (실행 시 설정하려면 귀찮으니까.)
```dockerfile
# 빌드 시점에 결정 (기본값 dev)
ARG PROFILE=dev 
# 이미지 안에 기본값으로 고정
ENV SPRING_PROFILES_ACTIVE=$PROFILE
```
- **주의! ARG와 ENV 모두 이미지 레이어에 데이터가 남는다. 따라서 민감 정보는 실행시에 동적으로 주입한다.**
  `docker run -e PASSWORD=secret my-spring-app`
## 완성된 예시 스크립트
```Dockerfile
# corretto 이미지로 시작한다.
FROM amazoncorretto:21

# app 디렉토리에서 작업하겠다. 그냥 정리용
WORKDIR /app

# 빌드 결과물을 이미지 내부로 복사한다.
# 이런 빌드 스크립트면, gradle 빌드를 개발자가 수동으로 하고 도커 빌드를 수행해야 한다.
COPY build/libs/*.jar coursepick-backend.jar

# 이 이미지는 8080포트를 쓰면 된다.
EXPOSE 8080

# 컨테이너 실행시 `java -jar coursepick-backend.jar` 명령을 실행한다.
ENTRYPOINT ["java", "-jar", "coursepick-backend.jar"]
```