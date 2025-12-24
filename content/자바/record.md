불변 데이터 클래스를 간단하게 작성하는 방법이다.
다음 메서드/기능들이 기본 구현된다.
- 모든 필드는 private final
- 전체 생성자
- getter
- equals / hashcode
- toString

```java
public class User {
    private final Long id;
    private final String name;
    private final String email;
    
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    public Long getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    
    @Override
    public boolean equals(Object o) { /* ... */ }
    
    @Override
    public int hashCode() { /* ... */ }
    
    @Override
    public String toString() { /* ... */ }
}

// Records
public record User(Long id, String name, String email) {}
```