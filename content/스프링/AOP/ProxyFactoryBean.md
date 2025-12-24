- [[JDK 동적 프록시]]는 **인터페이스가 필수**이다.
	- 인터페이스가 없는 클래스에 대해서는 CGLIB를 적용해줘야 한다.
- 또한 대상 클래스마다 **프록시 객체를 따로 생성**해서 빈 등록을 해야 한다.
	- 이는 '프록시를 어디에 붙일지'에 대한 개념이 분리되지 않았기 때문이다.
- 스프링에서는 이것을 나누어서 재사용 가능하도록 하였다.
	- `Advice` : 부가 기능 그 자체
	- `Pointcut` : 부가 기능을 어디에 붙일지
	- `Advisor` : 어드바이스+포인트컷, JDK 동적 프록시의 `InvocationHandler`와 비슷하다.
- 스프링에서는 포인트컷을 하나만 구현하면 이에 해당하는 부분에 자동으로 어드바이스를 적용해주는 `ProxyFactoryBean`을 제공한다.
- 또한 옵션으로 CGLIB를 사용할 수 있도록 한다.
- 하지만 여전히 타겟 객체를 지정해야 하는 문제가 있는데, 이는 [[DefaultAdvisorAutoProxyCreator]]로 해결한다.
### 구현 예시
**어드바이스**
```java
public class TransactionAdvice implements MethodInterceptor {  
  
    private final PlatformTransactionManager transactionManager;  
  
    public TransactionAdvice(PlatformTransactionManager transactionManager) {  
        this.transactionManager = transactionManager;  
    }  
  
    @Override  
    public Object invoke(final MethodInvocation invocation) throws Throwable {  
        final var transactionStatus = transactionManager.getTransaction(new DefaultTransactionDefinition());  
        try {  
            Object result = invocation.proceed();  
            transactionManager.commit(transactionStatus);  
            return result;  
        } catch (Exception e) {  
            transactionManager.rollback(transactionStatus);  
            throw new DataAccessException(e);  
        }  
    }  
}
```
**포인트컷**
```java
public class TransactionPointcut extends StaticMethodMatcherPointcut {  
  
    @Override  
    public boolean matches(final Method method, final Class<?> targetClass) {  
        return method.isAnnotationPresent(Transactional.class);  
    }  
}
```
**어드바이저**
```java
public class TransactionAdvisor implements PointcutAdvisor {  
  
    private final TransactionPointcut transactionPointcut;  
    private final TransactionAdvice transactionAdvice;  
  
    public TransactionAdvisor(TransactionPointcut transactionPointcut, TransactionAdvice transactionAdvice) {  
        this.transactionAdvice = transactionAdvice;  
        this.transactionPointcut = transactionPointcut;  
    }  
  
    @Override  
    public Pointcut getPointcut() {  
        return transactionPointcut;  
    }  
  
    @Override  
    public Advice getAdvice() {  
        return transactionAdvice;  
    }  
  
    @Override  
    public boolean isPerInstance() {  
        return false;  
    }  
}
```
### 사용 예시
```java
AppUserService target = new AppUserService(userDao, userHistoryDao);  
  
ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();  
proxyFactoryBean.setTarget(target);  
proxyFactoryBean.setProxyTargetClass(true);  // cglib 사용!
proxyFactoryBean.addAdvisor(new TransactionAdvisor(new TransactionPointcut(), new TransactionAdvice(platformTransactionManager)));  
  
AppUserService userService = (AppUserService) proxyFactoryBean.getObject();
```