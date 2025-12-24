먼저 null을 '버그'로 생각할 것인지, 아니면 '메시지'로 생각할 것인지 정해야 한다.
이 부분은 팀 컨벤션을 통해 정하면 될 것 같다.
## 관점1 : null은 버그다
null은 버그이므로 최대한 시스템 내에서 null을 제거해야 한다.
하지만 외부 시스템에 의존하는 영역에 대해서는 null 가능성을 고려할 수밖에 없다. 이 부분에서는 **완충 지대**를 만들어야 한다.
```java
class Controller {  
  
  Service service;  
  
  ResponseEntity<?> register(Request request) {  
    if (request.name() == null || request.password() == null) {  
      return ResponseEntity.badRequest().build();  
    }  
  
    service.register(request.name(), request.password());  
  
    return ResponseEntity.ok().build();  
  }  
}  
  
class Service {  
  
  void register(String name, String password) {  
    // 비즈니스 로직  
  }  
}  
  
record Request(  
    String name,  
    String password  
) {  
  
}
```
위 예시에서 `Controller`는 완충지대가 되며, 그 이하의 `Service`나 기타 도메인 모델들은 null 가능성을 고려할 필요가 없다.
## 관점2 : null은 메시지다
null은 메시지이므로 시스템의 어떠한 부분에서는 제거하고, 어떠한 부분에서는 사용할 수도 있다.
아쉽게도 자바에서 모든 참조타입은 null이 될 수 있으므로, null이 될 수 있는 것과 아닌 것을 구분할 수단이 필요하다.
최근에 사실상 표준이 된 [jspecify](https://jspecify.dev/docs/api/org/jspecify/annotations/package-summary.html)를 활용하면 이를 수행할 수 있다.
```java
class Controller {  
  
  Service service;  
  
  ResponseEntity<?> register(Request request) {
    service.register(request.name(), request.password());  
  
    return ResponseEntity.ok().build();  
  }  
}  
  
class Service {  
  
  void register(@Nullable String name, @Nullable String password) {
	if (name == null) throw new IllegalArgumentException();
	if (password == null) throw new IllegalArgumentException();
    // 비즈니스 로직
  }  
}  
  
record Request(  
    String name,  
    String password  
) {  
  
}
```
## 내 관점
jspecify의 등장도 그렇고, 결국 스프링은 null을 엄격하게 제어할 대상으로 보고 있다. (당연하다 NPE로 인한 문제가 크다.)
기존에는 관점1이 우세했던 것 같다. null을 표현할 수단이 정립되지 않았고, 이 때문에 개발자간에 생각 간극이 존재했던 것 같다.
이제는 jspecify를 사용하면서 관점2를 가져갈 수 있다. 물론 null을 무분별하게 받는 것은 여전히 위험하지만, 다음과 같은 코드에서는 빛을 발한다.
```java
@PersistenceCreator  
public Schedule(@Nullable Long id, String name) {  
  this.id = id;
  this.name = name;
}
```
관점1의 한계는 뚜렷하다. 아무리 열심히 완충지대를 만드려고 해도, 결국 생산성과 유지보수성의 목적으로 (이 경우 JPA) null을 허용할 수밖에 없는 경우가 발생한다.
이런 경우에는 관점2를 통해 이를 명확하게 나타내고, 더 안전한 코드를 작성할 수 있다.