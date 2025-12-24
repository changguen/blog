길고 장황한 타입 선언을 없애고, 추론하도록 한다.
```java
var map = new HashMap<String, List<Integer>>();
var list = new ArrayList<String>();
var count = 10;
```
적재적소에 사용하면 가독성을 향상시키지만, 아닐 수도 있기 때문에 호불호가 갈린다. (난 테스트 제외 불호)
정말 명확한 코드를 제외하고는 사용하면 악영향을 끼친다. (자바 개발자들은 이미 긴 타입에 꽤 익숙해졌다...)
```java
var result = googleDrive.getFile("topsecret_video"); // 무슨 타입??
```