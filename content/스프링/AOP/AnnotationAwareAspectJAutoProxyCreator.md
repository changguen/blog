- [[DefaultAdvisorAutoProxyCreator]]의 확장버전으로, `@Aspect`, `@Before`, `@After` 등의 애노테이션을 감지하여 작동한다.
	- 해당 애노테이션들을 파싱하여 `Advisor`로 변환한다.
	- 포인트컷에 해당하는 빈에 프록시를 적용한다.
- `@EnableAspectJAutoProxy` 또는 `@SpringBootApplication` 를 사용하면 자동으로 등록되는 빈이다.
- 기존 스프링의 언어들(`Advisor` 등)을 사용하지 않고, AspectJ의 것을 사용한다.
- 더 이상 `Advisor`를 구현할 필요없이, `@Aspect`와 함께 아스펙트를 구현하기 때문에, 편리하다.
- 'Spring AOP' 라고 하면 보통 이것을 떠올리면 된다.
```java
@Aspect
@Component
public class LoggingAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("메서드 호출 전: " + joinPoint.getSignature());
    }
    
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", 
                    returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("메서드 호출 후: " + joinPoint.getSignature());
    }
}
```