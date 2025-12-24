- 두 개념이 매우 비슷해서 헷갈리기 쉽다.
- 블로킹&논블로킹은 '제어 흐름의 위치'에 대한 개념이며
  동기&비동기는 '작업 완료 확인 방법'에 대한 개념이다.
- **블로킹** : 호출된 메서드가 자기 작업이 끝나기 전까지 제어권을 가진다.
- **논블로킹** : 호출된 메서드의 작업 여부와 상관없이 호출한 메서드가 제어권을 이어간다.
- **동기** : 작업의 결과 확인을 위해서는 작업이 완료될 때까지 기다린다.
- **비동기** : 작업의 결과를 콜백/이벤트/퓨쳐 등으로 능동적으로 받거나 풀링한다.
### 예시
- **블로킹 동기** : 일반적이고, 직관적이며 고전적인 방식
```java
System.out.println("[Start] 커피 주문");

// 1. 결과가 올 때까지 여기서 멈춤 (Blocking)
// 2. 리턴값을 변수에 직접 받음 (Synchronous)
String coffee = makeCoffee(); 

System.out.println("[Result] " + coffee + " 수령");
System.out.println("[End] 퇴장");
```
- **논블로킹 비동기** : 자원을 가장 효율적으로 사용하는 현대적 방식
```java
System.out.println("[Start] 커피 주문");

// 1. 비동기 작업 및 콜백 정의 (Asynchronous)
// 2. 설정만 하고 즉시 리턴됨 (Non-Blocking)
CompletableFuture.supplyAsync(this::makeCoffee)
		.thenAccept(coffee -> System.out.println("[Callback] " + coffee + " 수령"));

// 3. 커피가 안 나왔어도 즉시 다른 작업 수행
System.out.println("[Working] 다른 작업");
System.out.println("[End] 퇴장");
```
- **블로킹 비동기** : 비동기의 이점을 사용하지 못하는 비효율적인 상황
```java
System.out.println("[Start] 커피 주문");

// 1. 비동기 작업 및 콜백 정의 (Asynchronous)
CompletableFuture<Void> future = CompletableFuture.supplyAsync(this::makeCoffee)
		.thenAccept(coffee -> System.out.println("[Callback] " + coffee + " 수령"));

// 2. 비동기 작업이 끝날 때까지 메인 스레드를 강제로 멈춤 (Blocking)
future.join(); 

System.out.println("[End] 퇴장");
```
- **논블로킹 동기** : 논블로킹으로 즉시 리턴되며 다른 작업을 수행하지만, 작업 완료 여부를 계속 풀링하며 확인
```java
System.out.println("[Start] 커피 주문");

// 1. 작업 요청 후 즉시 Future 객체 받음 (Non-Blocking)
Future<String> future = executor.submit(() -> makeCoffee());

// 2. 결과가 준비되었는지 루프를 돌며 계속 확인 (Synchronous - Polling)
while (!future.isDone()) {
	System.out.println("[Working] 다른 작업");
}

// 3. 확인 완료 후 결과 조회
String coffee = future.get();
System.out.println("[Result] " + coffee + " 수령");
System.out.println("[End] 퇴장");
```