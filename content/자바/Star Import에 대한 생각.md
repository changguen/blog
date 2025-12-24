- Star Import를 제거하라는 것이 구글 스타일에 정의되어 있다.
- 그 이유는 대중적으로 다음과 같이 꼽힌다.
    - 명확성이 떨어진다. → 인정
    - 여러 패키지에 동일한 클래스명이 있다면 혼용될 수 있다. → 인정
        
        ```java
        import java.sql.*;
        import java.util.*;
        
        Date date; // 어느 Date?
        ```
        
- 근데, 그러면 명확하고 동일하지 않을 클래스는 괜찮을듯?

![[Star Import에 대한 생각_image.png]]