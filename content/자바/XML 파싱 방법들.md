- `DOM` : XML 전체를 메모리에 트리 구조로 올려서 처리
- `SAX` : XML을 앞에서부터 순차적으로 읽으며 처리 (스트리밍, 콜백 사용)
- `StAX` : XML을 앞에서부터 순차적으로 읽으며 처리 (스트리밍, pull 사용)
### 구현 차이
DOM은 전체를 한번에 로드하여, 원하는 부분을 마음대로 읽을 수 있다.
SAX는 콜백을 개발자가 넣어주면 파서가 push해서 호출되며, StAX는 파서를 개발자가 사용하면서 원할 때 pull하는 방식이다.
수도 코드를 통해 알아보자
```java
// DOM
void dom(Xml xml) {
	Document document = new Document(xml);
	
	Element element = document.readTag("root");
	System.out.println(element.getValue());
}

// SAX
void sax(Xml xml) {
	Parser parser = new Parser(
		xml,
		(content) -> System.out.println(content);
	)
	
	parser.run();
}

// StAX
void stax(Xml xml) {
	Parser parser = new Parser(xml);
	
	while (parser.hasNext()) {
		System.out.println(parser.next());
	}
}
```
위처럼 stax가 sax에 비해 코드가 순차적이며 쉬운 편이다.
### 장단점 비교
- `DOM` 
	- 전체를 메모리에 올리므로 자유롭게 움직이며 탐색할 수 있다.
	- 한번에 올리므로 메모리 사용량이 높다.
- `SAX`
	- 스트리밍하므로 메모리 사용량이 낮다.
	- 한번 넘어가면 다시 읽을 수 없다.
- `StAX`
	- `SAX`의 장단점을 그대로 가진다.
	- 추가적으로 `SAX`에 비해 가독성이 좋다.
### 번외 : JAXB
JAXB는 XML과 자바 객체 사이의 변환을 하는 방식이다.
```java
void jaxb(Xml xml) {
	User user = JAXB.parse(xml, User.class);
	
	String name = user.getName();
	System.out.println(name);
}
```
- 모두 로드하기 때문에 `DOM` 처럼 메모리 사용량이 높지만, 자바 객체를 다루기 때문에 사용하기 매우매우 쉽다.