- Executor의 비동기 작업 전후에 추가 로직을 넣어주는 인터페이스이다.
```java
public class DurationPrintTaskDecorator implements TaskDecorator {  
    @Override  
    public Runnable decorate(Runnable runnable) {  
        return () -> {  
            long start = System.currentTimeMillis();  
            try {  
                runnable.run();  
            } finally {  
                long duration = System.currentTimeMillis() - start;  
                System.out.println("Task executed in " + duration + " ms");  
            }  
        };  
    }  
}
```