- 보통 도메인 엔티티에 여러 애노테이션을 붙여서 JPA 설정 정보를 구성한다.
    
    ```java
    @Entity
    public class MemberWithAnno {
    
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      @Embedded
      private Email email;
      @Column(nullable = false, length = 20)
      private String nickname;
      @Embedded
      private Password password;
      @Enumerated(EnumType.STRING)
      private MemberStatus status;
      @OneToOne(cascade = CascadeType.ALL)
      private MemberDetail detail;
      
      // ...
    }
    ```
    
- 이런 설정 방법의 가장 큰 문제는 주요 정보(자바 코드)를 한 눈에 보기 어렵다는 것이다.
- 이런 설정 정보를 도메인 엔티티에서 분리하는 방법은 과거의 방식인 XML 설정 방식을 사용하는 것이다.

## XML로 JPA를 설정하기

- XML로 설정할 것이기 때문에 먼저 도메인 엔티티에서는 모든 JPA 엔티티를 제거한다.
    
    ```java
    public class MemberWithAnno {
    
      private Long id;
      private Email email;
      private String nickname;
      private Password password;
      private MemberStatus status;
      private MemberDetail detail;
      
      // ...
    }
    ```
    
- `resources/META-INF/orm.xml` 에 설정정보를 구성한다.
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <entity-mappings xmlns="https://jakarta.ee/xml/ns/persistence/orm"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                     xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence/orm
                     https://jakarta.ee/xml/ns/persistence/orm/orm_3_1.xsd"
                     version="3.1">
    		<!-- 위 부분이 XSD(XML의 문법)을 설정하는 부분이다. -->
    		
        <access>FIELD</access>
    
        <entity class="dompoo.splearn.domain.member.Member">
            <attributes>
                <id name="id">
                    <generated-value strategy="IDENTITY"/>
                </id>
                <basic name="nickname">
                    <column length="20" nullable="false"/>
                </basic>
                <basic name="status">
                    <enumerated>STRING</enumerated>
                </basic>
                <one-to-one name="detail">
                    <cascade>
                        <cascade-all/>
                    </cascade>
                </one-to-one>
                <embedded name="email"/>
                <embedded name="password"/>
            </attributes>
        </entity>
        
        // ...
    
    </entity-mappings>
    ```
    

## XML 구성 방식의 장단점과 내 생각

- 장점
    - **도메인 엔티티의 주요 로직이 한 눈에 들어온다.**
- 단점
    - 설정하기 어렵다. → 공부하자.
    - 도메인 엔티티의 어떤 정보를 수정할 때, orm.xml도 수정해야 한다는 사실을 잊기 쉽다.
    → 먼저 애노테이션으로 설정을 한 후에, 이를 orm.xml에 옮기는 방식을 사용하면 해결할 수 있다.
- 내 생각
    - JPA 애노테이션을 사용한다고 해서 도메인 엔티티가 JPA에 의존하는 것은 아니다. 애노테이션은 어디까지나 주석이기 때문이다. **문제는 오로지 가독성에 있다.**
    - JPA 애노테이션이 주요한 정보를 담는 경우도 존재할 수 있을 것 같다. 예를 들어 `nullable = false` 설정은 필수적인 속성이라는 정보를 전한다.
    - **결국 JPA 애노테이션은 정보를 주지만, 그 정보가 과하다는 것이 문제이다.**

## 행위가 우선이 되는 코드와 속성이 우선이 되는 코드

- 작은 코드베이스에서는 모든 속성을 기억할 수 있는 것 같다.
    - ex) Member 엔티티의 name은 필수고, 10자리 이하이다.
- 하지만 점점 복잡해질수록 기억에 남는 것은 행위이다.
    - ex) Member는 가입될 수 있고, 탈퇴할 수 있다.
- 따라서 코드가 성장하며 우리는 **행위를 기억해야 하고 가꾸어 나가야한다.**
- 이런 행위를 주요하게 여길 수 있는 방식은 XML 설정 방식인 것 같다.