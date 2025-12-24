클래스의 계층 구조를 코드적으로 제한하는 방법이다.
**기능**
- 허용된 서브 클래스만 상속 가능하도록 한다.
- 더 이상 상속 불가능하도록 한다.
- 상속 제한을 해제한다.
**장점**
- switch 문 등에서 타입이 명확하게 제한되므로, default가 필요없다.
- 도메인 모델의 의도를 명확히 표현할 수 있다.
```java
// 허용된 서브클래스만 상속 가능
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
}

public final class Circle extends Shape {
    private final double radius;
    // ...
}

public final class Rectangle extends Shape {
    private final double width;
    private final double height;
    // ...
}

public non-sealed class Triangle extends Shape {
    // non-sealed: 하위에서 자유롭게 상속 가능
}
```
