먼저 식과 문의 차이를 알아야 한다.
- **문**은 작업이 종료된 완전한 문장이다.
  ex) `var greetings = "Hello World";`
- **식**은 작업이 끝난 것이 아닌, 값을 만들어내는 것이다.
  ex) `1+1`, `"Hello World"`

기존의 **switch 문**은 종료된 완전한 문장이었다.
```java
String result;

switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        result = "6글자";
        break;
    case TUESDAY:
        result = "7글자";
        break;
    default:
        result = "기타";
        break;
}
```

하지만 **swtich 표현식**은 이 switch 문과 비슷한 문법을 지니면서, 값을 만드는 식으로 작동한다.
```java
String result = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> "6글자";
    case TUESDAY -> "7글자";
    default -> "기타";
};
```

### 문법적 차이점
- `:` 대신 `->`를 쓴다.
- `break`가 필요없다. (한번 충족되면 종료된다.)
- 모든 케이스를 처리해야 한다.
	- 열거형의 경우엔 모든 열거형 상수를 처리해야 한다.
	- 기타 타입의 경우엔 default가 필수다.
- 여러 줄인 경우에는 중괄호를 열고 `return`이 아닌 `yield`를 통해 값을 반환해야 한다.
```java
int result = switch (operator) {
    case '+' -> {
	    log.info("덧셈!");
	    int res = num1 + num2;
	    yield res;
	};
    default -> throw new IllegalArguemtnException();
};
```
