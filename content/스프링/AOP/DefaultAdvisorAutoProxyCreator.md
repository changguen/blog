- [[ProxyFactoryBean]]을 사용해도 여전히 타겟을 지정해야 한다.
- `DefaultAdvisorAutoProxyCreator`를 사용하면 스프링 컨테이너 초기화 시점에 모든 빈을 검사하여 적용 가능한 Advisor가 있으면 **자동으로 프록시를 생성한다.**
```java
@Configuration  
public class AopConfig {  
  
    @Bean  
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {  
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();  
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);  // cglib 사용
        return defaultAdvisorAutoProxyCreator;  
    }  
  
	// 직접 구현한 어드바이저
    @Bean  
    public TransactionAdvisor transactionAdvisor(PlatformTransactionManager platformTransactionManager) {  
        return new TransactionAdvisor(new TransactionPointcut(), new TransactionAdvice(platformTransactionManager));  
    }  
}
```
- 위처럼 하기만 하면 마법같이 어드바이저가 적용된다.
- 다만 더 편리한 [[AnnotationAwareAspectJAutoProxyCreator]] 방식이 존재하기 때문에, 이렇게 직접 등록하는 일은 없다.