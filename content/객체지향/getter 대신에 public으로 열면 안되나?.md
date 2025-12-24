```java
class Member {
	private final String name;
	private final int age;
	
	// 생성자
	
	public String getName() {
		return this.name;
	}
	
	public int getAge() {
		return this.age;
	}
}
```
- 위와 같은 클래스를 작성하고 나면 결국 getter를 추가해서 코드량을 늘릴거면, 그냥 private을 public으로 열어두면 안되나? 싶은 생각이 들었다.
```java
class Member {
	public final String name;
	public final int age;
	
	// 생성자
}
```
- 내 생각에 '게터'라는 말을 쓰고, 그렇게 `get~`처럼 구현할거면 진짜 public으로 열어두어도 된다.
	- 어차피 final이라 외부에서 수정될 일도 없기 때문이다!
- 포인트는 저런 **단순 반환 기능도 또한 하나의 기능이라고 생각**하는 것이다.
```java
class Member {
	private final String name;
	private final int age;
	
	// 생성자
	
	public String name() {
		return this.name;
	}
	
	public int age() {
		return this.age;
	}
	
	public String nickname() {
		if (this.nickname == null) return name + "뿅";
	}
}
```
- 외부의 입장에서는 해당 메서드가 단순 필드 반환(게터)인지 복잡한 기능을 수행하는 것인지 알 필요가 전혀 없다.
	- 단지 회원은 자신의 이름/나이/별명을 응답할 수 있다는 것만 인지하면 된다.
- **따라서 '게터'라고 부르면 안되며, 메서드명도 `age()` 처럼 작성해야 한다.**
  **그렇게 하지 않을 것이라면 그냥 public으로 열어라.**