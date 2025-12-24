AOP가 결국 해당 객체를 감싸는 [[AOP와 프록시의 관계|프록시]] 방식으로 작동한다는 것을 생각하면 유추 가능하다.
## Self-Invocation 문제
: 같은 객체 내에서 다른 메서드를 호출하면 AOP를 타지 않는다.
```java
@Service
public class Service {
	
	public a() { }
	
	@Transactional
	public b() { }
}
```
- 정상적인 상황 : proxy#b -> bean#b -> 예외 발생 -> bean#b -> proxy#b (롤백처리)
- 자기호출 문제 : proxy#a -> bean#a -> bean#b -> 예외 발생 -> bean#b -> bean#a -> proxy#a (프록시의 a메서드에는 롤백 처리가 없다.)
**해결 방법**
- 비즈니스 로직의 시작점에서 항상 트랜잭션을 시작한다.
- 트랜잭션 필요 여부에 따른 적절한 객체 분리를 진행한다.
- Self-Injection
## 접근 제어자 제약
: AOP는 private 메서드를 감싸지 못한다.
최신 스프링에서는 CGLIB를 사용하며, 타겟 클래스를 상속받는 방식을 사용하는데, 따라서 private 메서드는 오버라이딩되지 않아 문제가 발생한다.
![[Pasted image 20251201161257.png]]
## final 키워드 제약
: AOP는 CGLIB (상속)을 기반으로 작동한다.
따라서 final 클래스는 상속할 수 없기에, 트랜잭션 적용이 불가능하다. (IDE에서 오류가 발생하기도 한다.)
![[Pasted image 20251201161214.png]]
## 체크 예외
: 스프링에서 롤백은 런타임 예외로 한정된다.