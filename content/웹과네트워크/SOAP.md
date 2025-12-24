- Simple Object Access Protocol
- XML 기반의 구조화된 메시징 프로토콜이다.
- **엄격한 표준과 보안 기능을 제공한다.**
- 주로 HTTP 위에서 작동하지만, 프로토콜 독립적이다.
- `WSDL` : 서비스의 기능, 메시지 형식, 엔드포인트들을 명시적으로 정의하는 것이다. 클라이언트가 서비스를 자동으로 이해하고 사용할 수 있도록 한다.
### 통신 프로세스
#### 서버 측에서 제공할 웹 서비스에 대한 WSDL을 작성한다.
```xml
<!-- 예시: 계좌 조회 서비스 WSDL -->
<definitions name="BankService"
  targetNamespace="http://bank.example.com/"
  xmlns:tns="http://bank.example.com/">
  
  <!-- 메시지(입출력) 타입 정의 -->
  <message name="GetBalanceRequest">
    <part name="accountNumber" type="xsd:string"/>
  </message>
  
  <message name="GetBalanceResponse">
    <part name="balance" type="xsd:decimal"/>
  </message>
  
  <!-- 오퍼레이션 정의 -->
  <portType name="BankServicePortType">
    <operation name="GetBalance">
      <input message="tns:GetBalanceRequest"/>
      <output message="tns:GetBalanceResponse"/>
    </operation>
  </portType>
  
  <!-- 바인딩 및 엔드포인트 -->
  <binding name="BankServiceBinding" type="tns:BankServicePortType">
    <soap:binding transport="http://schemas.xmlsoap.org/soap/http"/>
    <operation name="GetBalance">
      <soap:operation soapAction="http://bank.example.com/GetBalance"/>
    </operation>
  </binding>
  
  <service name="BankService">
    <port name="BankServicePort" binding="tns:BankServiceBinding">
      <soap:address location="http://bank.example.com/services/bank"/>
    </port>
  </service>
</definitions>
```
#### 클라이언트에서 WSDL을 획득한다.
`GET /bank.example.com/services/bank?wsdl`
#### 클라이언트에서 WSDL을 기준으로 요청을 구성하고 보낸다.
```xml
POST /services/bank HTTP/1.1
Host: bank.example.com
Content-Type: text/xml; charset=utf-8
Content-Length: xxx
SOAPAction: "http://bank.example.com/GetBalance"

<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope 
  xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:bank="http://bank.example.com/">
  
  <!-- SOAP Header (선택사항) -->
  <soap:Header>
    <bank:AuthToken>abc123xyz</bank:AuthToken>
  </soap:Header>
  
  <!-- SOAP Body (필수) -->
  <soap:Body>
    <bank:GetBalanceRequest>
      <bank:accountNumber>1234-5678-9012</bank:accountNumber>
    </bank:GetBalanceRequest>
  </soap:Body>
  
</soap:Envelope>
```
#### 서버에서 이를 처리한다.
- XML 파싱
- 헤더 처리
- Body 추출
- 메서드 호출
### REST API와의 비교
- `REST`가 '설계철학'인 것과는 다르게, `SOAP`는 엄격한 규칙이다.
- `REST`에서 Method는 중요한 정보인 것과는 다르게, `SOAP`에서는 보통 POST로만 요청하고, 정보는 XML에 담긴다.
- `REST`가 리소스 중심 설계인 것과는 다르게, `SOAP`는 액션 중심 설계이다. (약간 RPC 스타일)
- `REST`가 웹의 장점을 활용하면서 단순하게 사용하는 것에 비해, `SOAP`는 모든 것을 표준화하기 때문에 엔터프라이즈에 좀 더 적합하다. (정말? 생산성은 어따두고.)