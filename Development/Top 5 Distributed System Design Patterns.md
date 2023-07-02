https://medium.com/better-programming/top-5-distributed-system-design-patterns-ae9482f49128

## 1. CQRS

복잡한 update 처리를 하게 된다면(10개의 테이블을 조인한다던지..) 너무나 많은 락을 잡게 된다.
쓰기와 읽기의 책임을 분리하여, 직관적으로 구성할 수 있다.

command = 쓰기 작업
query = 읽기 작업

쓰기전용 DB와 읽기전용 DB를 구축하여, 쓰기전용 DB에 쓰여진 데이터를 어떻게 읽기전용 DB로 복제하는지에 대한 이야기가 주를 이루는 것 같다.
Event Sourcing 을 통해 DB sync 작업을 수행한다. 복제지연이 있을 수 있는데, 해당 문서에서는 더 이야기하지는 않았다.

### Materialized view
Materialized view 를 따로 구성하여 읽기 전용으로 사용한다.

### Instagram 예시

PostrgreSQL은 사용자 정보들을 갱신/저장하는 용도로 사용되고
Cassandra의 경우에는 저장된 유저의 Stories를 보여주는 Read에 특화된 데이터를 보여준다.

https://medium.com/design-microservices-architecture-with-patterns/cqrs-design-pattern-in-microservices-architectures-5d41e359768c

## 2. 2PC

"코디네이터(트랜잭션 관리자)" 를 통해서 원자적 트랜잭션을 제공한다.

모든 노드에서 데이터를 쓸 수 있는 상황인지를 Phase 1 에서 확인하는 과정을 거친 뒤,
Phase 2 에서 데이터를 쓰거나 롤백하는 과정을 거친다.

코디네이터에 의존적인 것과 블록킹이 된다는 점이 단점인 것으로 확인된다. (성능이슈가 있음)

https://kadensungbincho.tistory.com/125

## 3. SAGA

롤백이 발생해야하는 경우, 보상처리를 통해 원상태를 복구하는 기법

각각의 서비스들을 각자의 로컬 트랜잭션을 통해 처리를 수행하고, 연계된 서비스가 모두 성공하지 못 했다면
데이터를 삭제하거나 수정하는 이벤트(보상 트랜잭션)를 발행하여 다른 서비스들을 트랜잭션 수행 이전 상태로 복원시킨다.

이벤트를 통해, 해당 트랜잭션이 성공되었는지 실패했는지를 다른 서비스들에 알릴 수 있어야 한다.

2PC와 비슷하다고 느낄 수 있지만, SAGA에서는 코디네이터가 없는 방식으로도 구현할 수 있다. (decentralization)

https://azderica.github.io/01-architecture-msa/


## 4. RLBS

하나의 Load Balance 아래에 여러 개의 Replica 가 존재하여 서비스를 제공하는 형식?

## 5. Sharded Services

특정 서비스들은 특정 요청들만 처리할 수 있도록 기능 자체를 서비스레벨에서 Sharding 해놓는 방식

1. 캐시된 기능들만 받는 서비스
2. 중요요청들만 받는 서비스
3. 이외의 요청들을 받는 서비스

LB에서 적절히 분산해줄 수 있어야 한다. 이정도면 L7 써야하지 않나