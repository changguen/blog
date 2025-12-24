- `InvocationHandler`와 `Proxy`를 통해서 프록시 객체를 만들어내는 방법이다.
- **대상에 인터페이스가 무조건 존재해야 한다.**
- `InvocationHandler`를 통해 프록시 로직을 작성하고, `Proxy`를 통해서 프록시 객체를 생성한다.
- 이와 다르게, `CGLIB` 라는 라이브러리는 바이트코드 조작을 통해 인터페이스 없이도 프록시 객체를 만들어낸다.
### 대상 객체들
```java
public interface UserService {  
    User findById(final long id);
}
```

```java
public class AppUserService implements UserService {  
  
	// ...

    @Transactional  
    public User findById(final long id) {  
        return userDao.findById(id);  
    }  
}
```
### 프록시
```java
public class TransactionHandler implements InvocationHandler {  
  
    private final UserService target;  
    private final PlatformTransactionManager transactionManager;  
  
    public TransactionHandler(UserService target, PlatformTransactionManager transactionManager) {  
        this.target = target;  
        this.transactionManager = transactionManager;  
    }

     @Override  
    public Object invoke(final Object proxy, final Method method, final Object[] args) {  
        final var transactionStatus = transactionManager.getTransaction(new DefaultTransactionDefinition());  
        try {  
            Object result = method.invoke(target, args);  
            transactionManager.commit(transactionStatus);  
            return result;  
        } catch (Exception e) {  
            transactionManager.rollback(transactionStatus);  
            throw new DataAccessException(e);  
        }  
    }  
}
```
### 프록시 객체 생성
```java
AppUserService target = new AppUserService(userDao, userHistoryDao);  
UserService userService = (UserService) Proxy.newProxyInstance(  
        target.getClass().getClassLoader(),  
        target.getClass().getInterfaces(),  
        new TransactionHandler(target, platformTransactionManager)  
);
```