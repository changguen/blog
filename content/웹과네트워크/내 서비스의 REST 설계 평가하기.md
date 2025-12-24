- [[REST]]를 공부하다보니까 감이 안잡혀서 내 서비스의 API들을 직접 평가해보려고 한다.
### 구글 드라이브와 코스 싱크 맞추기
**현재**
```text
POST /admin/courses/sync?adminToken=123123
```
**평가**
- 인증 토큰을 쿼리 파라미터로 전달하는 것은 보안 취약점이다.
	- 서버 로그에 평균으로 기록됨
	- 브라우저 히스토리에 저장됨
	- URL 공유시 토큰이 노출됨
	-> Authorization 헤더 사용
- `sync`가 동사로, RPC 스타일에 가까움
	- 싱크를 맞춘다는 개념의 적절한 HTTP Method 사용
**결과**
```
PUT /admin/courses

{
	"adminToken": "123123"
}
```
### 현재 위치와 가까운 코스 찾기
**현재**
```
GET /courses?mapLat=123&mapLng=123&userLat=123&userLng=123&scope=1000&page=1
```
**평가**
- 전체적으로 REST하게 잘 구성되어 있다.
- 다만, mapLat/Lng와 userLat/Lng로 4개가 나뉘어져 있는데, 파라미터를 그룹화하면 더 명확할 수 있다.
- 또한 카멜케이스 대신에 케밥케이스를 사용해야 한다.
**결과**
```
GET /courses?map-position=123,123&user-position=123,123&scope=1000&page=1
```
### 한 코스와 가장 가까운 좌표 찾기
**현재**
```
GET /courses/123/closest-coordinate?lat=123&lng=123
```
**평가**
- 코스 하위의 여러 좌표 중에 하나를 응답하므로, `coordinates` 라는 이름이 URL에 들어가는게 좋다.
- 다만 위와 비슷하게 lat/lng를 묶어주면 좋을 것 같다.
**결과**
```
GET /courses/123/coordinates/closest?target-position=123,123
```
### 한 코스까지의 길찾기
**현재**
```
GET /courses/123/route?startLat=123&startLng=123
```
**평가**
- 위와 동일하게 lat/lng를 묶어야 한다.
**결과**
```
GET /courses/123/route?start-position=123,123
```
### 특정 ID들의 코스 찾기 (즐겨찾기 검색용)
**현재**
```
GET /courses/favorites?courseIds=1,2,3
```
**평가**
- 해당 API의 용도 자체는 즐겨찾기 검색용이 맞으나, 행위는 그냥 courseId들을 기준으로 검색하는 것이다.
  -> favorites 같은 단어를 쓸 필요가 없다.
- 케밥케이스를 사용한다.
**결과**
```
GET /courses?ids=1,2,3
```