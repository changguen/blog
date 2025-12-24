> GC란 힙 메모리에서 더이상 사용하지 않는 객체를 식별하고 자동으로 제거하는 프로세스이다.
## 중요 개념 : 도달 가능성
- GC Root : 외부에서 객체를 참조하는 시작점
  로컬 변수, static 변수 등이 된다.
- Reachable : GC Root를 시작으로 참조 사슬을 통해 도달 가능한 객체
- Unreachable : 도달할 수 없는 객체
## 기본 프로세스 : Mark and Sweep
- Mark : GC Root 부터 시작해서 모든 객체의 도달 가능성을 식별한다.
- Sweep : Unreachable 객체를 제거한다.
## JDK 9 부터의 기본 GC : G1GC
> Garbage-First Garbage Collector
- 전체 힙을 지역 단위로 나누고, 관리한다.
- 각 지역은 Eden, Survivor, Old 등의 역할이 유동적으로 변한다.
- 모든 지역을 뒤지는게 아니라, 가비지가 많은 지역부터 우선적으로 선별/정리한다. (Garbage-First)
	- 가비지는 그대로 두고, 생존한 객체만 다른 지역으로 이동시켜준다.