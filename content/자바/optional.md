다른 시스템의 문제 상황을 나타내는 응답값은 자바에서 너무 많은 형태가 가능하다.
- 특정 값 (0, -1)
- null
- 런타임 예외, 체크 예외
```java
public String getUserCityName(Long userId) {
	User user;
	try {
		// 널이 넘어올까? 예외가 발생할까?
		user = userRepository.findById(userId);
	}
	catch (Exception e) {
		return "Unknown"; // 내 시스템은 무슨 응답값을 줘야 할까?
	}
    
    // 복잡하다.
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            City city = address.getCity();
            if (city != null) {
                return city.getName();
            }
        }
    }
    return "Unknown";
}
```
이 모호함을 해결하여, 명확한 코드를 작성하고 버그를 방지하는 것이 **Optional**의 목적이다.
```java
public Optional<String> getUserCityName(Long userId) {
	Optional<User> user = userRepository.findById(userId);
	return user
		.map(User::getAddress)
        .map(Address::getCity)
        .map(City::getName);
}
```