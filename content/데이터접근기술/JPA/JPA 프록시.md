- find 한 객체는 프록시가 아니다.
- 그 객체가 LAZY 하게 들고 있는 다른 연관관계가 프록시이다.

```java
class Member {
  // Lazy
	List<Reservation>
	
	// EAGER
	List<Friend>
	
	// Lazy
	Setting
	
	// EAGER
	Address
}
```

- 위와 같은 객체를 find 하면 아래와 같이 된다.

```java
class Member { // <- find 한 객체는 그대로이다.
  // Lazy
	PersistentBag // <- List는 PersistentBag
	
	// EAGER
	List<Friend>
	
	// Lazy
	HibernateProxy // <- 1개 짜리는 HibernateProxy
	
	// EAGER
	Address
}
```

![[JPA 프록시_image.png]]

- 해당 객체들의 메서드(특히 SQL이 필요한 메서드)를 호출한 순간 Lazy-Loading 된다.