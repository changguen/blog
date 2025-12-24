자바에서 컬렉션 데이터를 처리하는 방식은 for/if 문으로 한정되었다.
```java
for (Person person : people) {
    if (person.getAge() >= 20) {
        result.add(person.getName());
    }
}
Collections.sort(result);
```
하지만 코드를 이렇게 작성하면 매우 복잡해진다.
**스트림 API**는 이런 코드를 선언적으로 바꾸어 간결하고, 가독성 좋게 만들어준다.
```java
List<String> result = people.stream()
    .filter(p -> p.getAge() >= 20)
    .map(Person::getName)
    .sorted()
    .collect(Collectors.toList());
```
추가적으로 다음 장점도 지닌다.
- [[스트림 API의 지연 연산|최종 연산 전까지 실행이 안되므로, 성능 최적화가 된다.]]
- 원본을 수정하지 않는다.