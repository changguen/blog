- MongoDB를 EC2에 하나만 띄워서 사용하면 그것은 SPOF가 된다. 좋지 않다!
    - 요청이 너무 많아 CPU 레이트가 뛰어버려서 멈추는 경우
    - EC2의 설정등을 수정하기 위하여 잠시 stop 시킬 필요가 있는 경우
    - …
- 여기서 Replica Set이 등장한다.

## Replica Set

> 여러 대의 MongoDB가 동일한 데이터를 공유하여, 하나가 실패하면 다른 것이 이어받도록 한다.
> 
- 기본 구조
    - **Primary** : 쓰기/읽기 등의 작업을 처리한다.
    - **Secondary** : Primary의 데이터를 실시간 복제하는 백업
- **FailOver** : Primary가 죽으면 Secondary가 자동으로 Primary로 승격되어, SPOF를 해결
- **Oplog** : **Primary에서 일어나는 모든 변경을 로그로 기록, Secondary에서는 이를 읽어서 반영한다.**

![[MongoDB에서 Replica Set_image.png]]

### 선출 메커니즘

> Primary에 문제가 생겼을 때, Secondary 중 하나가 Primary가 되는 자동화된 프로세스이다.
> 
- Primary 후보 노드의 작동 상태/데이터 동기화 정도/선출 우선순위 등을 기준으로 각 Secondary가 투표한다.
- 과반수가 찬성하면 Primary로 선출된다.
- 깊게 들어가면 복잡하다.

### Write Concern (쓰기 안정성 수준)

> 클라이언트가 쓰기 쿼리시 ‘얼마나 안전하게 쓰기를 보장할 것인가’를 지정할 수 있다.
> 
- 옵션 예시
    - Primary에만 기록되면 OK (빠르다)
    - 다수 노드에 기록되면 OK (비교적 느리다)
- 성능과 안정성의 트레이드오프를 쿼리 시점에 조정할 수 있다.

```json
db.collection.insertOne(
	{ name: "dompoo" },
	{ writeConcern: { w: "majority", j: true } }
)
```

### Read Concern (읽기 안정성 수준)

> 클라이언트가 읽기 쿼리시 ‘얼마나 안전하게 저장된 데이터를 읽을 것인가’를 지정할 수 있다.
> 
- 옵션 예시
    - 가장 최신의 데이터
    - 다수의 노드가 커밋되었다는 것을 보장하는 데이터
    - Primary에 커밋된 데이터 (강력)

### Read Preferences (읽기 우선 순위)

> 클라이언트가 읽기 쿼리하는 대상의 우선순위를 지정해줄 수 있다.
> 
- 옵션 예시
    - 항상 Primary에서 읽기 (일관성 ↑)
    - Primary에서 최대한 읽고, 불가 시 Secondary
    - 항상 Secondary
    - Secondary에서 최대한 읽고, 불가 시 Primary
    - 가장 네트워크 지연이 낮은 노드
- 성능의 일관성의 트레이드오프이다.

## 직접 실습하기

---