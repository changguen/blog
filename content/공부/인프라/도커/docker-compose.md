> 여러개의 컨테이너를 한번에 관리할 수 있도록 도와주는 도구

- yml 기반으로 여러 컨테이너를 한번에 띄우고 서로 연결할 수 있다.
- 각 컨테이너는 이름을 갖으며, 이 이름으로 서로를 참조할 수 있다.
- 각 컨테이너는 다음 값들을 설정할 수 있다.
	- 이미지
	- 컨테이너 이름
	- 포트 포워딩 : 호스트 포트와 컨테이너 포트를 연결
	- [[도커 볼륨|볼륨]]
	- [[도커 네트워크|네트워크]]
	- 의존성 : 다른 컨테이너가 먼저 실행되어야 하는 경우
	- 재시작 정책 (`restart`) : 컨테이너가 중지된 경우의 정책
		- no : 재시작 안함 (기본)
		- always : 항상 재시작
		- on-failure : 에러로 종료됐을 경우 재시작
		- unless-stopped : 수동으로 중지하기 전까지 재시작
- 여러 컨테이너를 띄울 때 뿐만 아니라, 1개를 띄우더라도 표현력이 좋아서 자주 사용한다.
## 예시(1개 컨테이너)
```yml
name: coursepick-local  
services:  
  
  mongodb:  
    image: mongo:8.0.12  
    container_name: mongodb  
    environment:  
      - MONGO_INITDB_ROOT_USERNAME=root  
      - MONGO_INITDB_ROOT_PASSWORD=secret  
    ports:  
      - "27017:27017"  
    volumes:  
      - mongodb_data:/data/db  
      - mongodb_config:/data/configdb  
  
volumes:  
  mongodb_data:  
  mongodb_config:
```