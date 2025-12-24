- Redis와의 연결, 쿼리 등을 추상화해서 다룰 수 있게 해준다.
- 다른 여타 `Spring Data` 계층들처럼 `RedisTemplate`을 제공해준다. 이를 통해 접근하면 된다.
- 또는 `Spring Cache`의 애노테이션과 통합하는 방법을 활용해도 된다. (더 추상적이며 간편하다.)
- 필요해지면 더 찾아보고 사용하면 될 듯!
### RedisTemplate 사용 예시
```java
@Service
@RequiredArgsConstructor
public class RedisStringService {
    
    private final RedisTemplate<String, Object> redisTemplate;
    
    public void setValue(String key, String value) {
	    redisTemplate.opsForValue().set(key, value);
    }

    public Boolean setIfAbsent(String key, String value) {
        return redisTemplate.opsForValue().setIfAbsent(key, value);
    }
    
    public String getValue(String key) {
        return (String) redisTemplate.opsForValue().get(key);
    }
```
### Spring Cache 통합
**캐시 활성화**
```java
@Configuration
@EnableCaching  // 캐싱 기능 활성화
public class CacheConfig {
    
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        // 기본 캐시 설정
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))  // 기본 TTL 1시간
            .disableCachingNullValues()  // null 값은 캐싱하지 않음
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
        
        // 캐시별 개별 설정
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        
        // 사용자 캐시: 30분 TTL
        cacheConfigurations.put("users", 
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))
        );
        
        // 상품 캐시: 10분 TTL
        cacheConfigurations.put("products", 
            RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
        );
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```
**캐시 사용**
```java
@Service
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    
    // 기본 사용
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        System.out.println("DB 조회: " + id);
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }
    
    // 복합 키 사용
    @Cacheable(value = "users", key = "#name + '_' + #age")
    public List<User> findByNameAndAge(String name, int age) {
        return userRepository.findByNameAndAge(name, age);
    }
    
    // 객체를 키로 사용
    @Cacheable(value = "userSearch", key = "#request.toString()")
    public List<User> searchUsers(UserSearchRequest request) {
        return userRepository.search(request);
    }
}
```