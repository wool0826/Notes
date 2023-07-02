https://medium.com/wix-engineering/event-driven-architecture-5-pitfalls-to-avoid-b3ebf885bdb1


뭔가 몰랐던 것을 알게되지않을까 해서 읽으면서 정리해보았는데,,,, plasma 에서 하던 작업들이 대부분 포함되어있네요.
복습의... 느낌으로 봐주시면 될 것 같습니다.


## 개요

![화면 캡처 2022-11-27 212524.png](/files/0a710dbd-83f9-1069-8184-b90e3ae10e7d)


EDA는 분산 마이크로서비스 환경에 잘 맞고 유용한 아키텍처
"브로커 중개자" 라는 것을 통해 결합도가 낮은 아키텍쳐, 쉬운 확장성, 더 높은 유연함을 제공할 수 있다.

하지만 정확히 구축하는 것이 기존의 request-reply client-server 아키텍처보다 어렵다.

Wix 라는 곳에서 플라즈마와 비슷하게 기존의 구조를 EDA 구조로 변경하는 작업을 하고 있는데 "5개의 함정" 이 있다고 판단하였다.

이에 대한 내용을 공유한다.

## 1. 원자성없이 DB에 쓰고 이벤트 발행하기

가장 일반적으로 고려하는 케이스인 것 같다.
A라는 시스템에서 성공한 뒤, B 시스템에 결과를 전달하고 반영해야하는데 이 작업은 원자적이지 않다.

왜냐하면 B의 DB가 죽거나, 메시징 큐가 죽거나 해서 메시지가 전달되지 않는다면 A와 B의 데이터가 일관적이지 않게 되기 때문이다.

아래 예시를 보면 알 수 있지만, "결국은 상태를 맞게 만든다" 에 초점이 맞춰져있다.

### 1-1. 메시지를 어떻게던지 보내게 구성한다.

이 글에서는 자기들이 구축한 GreyHound 라는 플랫폼을 이용하여, 결국 Kafka로 메시지를 보내게 하였다고 한다.
메시지가 순서대로 처리되지 않거나, 지연이 있을 수 있다.

### 1-2. Debezium Kafka source connector 를 사용한다.

Debezium Kafka Source Connector 는 MySQL의 binlog 를 통해 DB의 change event를 추적하여 메시지를 발행할 수 있다고 한다.
이 라이브러리도 Eventually Consistency를 만족한다.

> binlog 를 읽어서 발행하는 것의 장점
> 
> 트랜잭션이 커밋이 되었다면 binlog에 작업이 써진다. => 해당 데이터가 커밋이 되었음을 보장할 수 있다.
> 여기서 중요한 점은 **트랜잭션을 보장한다** 는 점이다.

## 2. Event Sourcing 을 아무곳에서나 사용한다.

![화면 캡처 2022-11-27 212650.png](/files/0a710dbd-83f9-1069-8184-b90e3ae00e7c)

데이터를 변경하는대신, 변경하는 Event를 저장하는 방식을 Event Sourcing 패턴이라고 한다.
서비스에서는 해당 Event를 "replay" 해서 최종상태를 알아낼 수 있다.
이런 Event는 Event Bus로 발행될 수 있고, 다른 서비스에서도 해당 Event를 Replay 해서 최종상태를 알아낼 수 있다.

이런 Event Sourcing "기존의 객체를 변경하는 CRUD 서비스"보다 복잡하다는 것이 단점.

- 장점
    - 신뢰할 수 있는 감사 로그
    - 동일한 데이터에 대해 언제든지 엔티티 상태를 파악할 수 있는 기능
    - 동일한 데이터를 통해 여러가지 뷰를 생성할 수 있는 기능
- 단점
    - 복잡하다.
    - Snowflake nature (번역은 해 놓았지만 뭔 이야긴지 이해 못 했습니다.)
        - CRUD ORM 솔루션들과 달리, 일반적인 사용상황에 적절한 snapshot 기능과 읽기 최적화 기능을 가지는 범용 라이브러리나 프레임워크의 개발이 힘들다.?
    - Eventually Consistency 만을 지원함.
        - read after write 의 경우에는 문제가 발생함. (즉, 변경사항을 바로 읽지 못하는 경우가 존재한다는 의미인듯)

### 2-1. Event Sourcing 대체 - CRUD + CDC

CQRS 관점에서 봤을 때도 적절하게 구성할 수 있는 구조인 것으로 이해했습니다.

DB 상에는 CRUD 기반으로 작업하되, CDC를 통해 Change Event를 이벤트로 발행한 뒤 해당 이벤트를 반영하는 View 를 만들면
Update 하는 DB와 Search 하는 DB가 따로 존재하게 되고, 유연하고 덜 복잡한 구조를 만들 수 있다.

추가적인 view가 필요한 경우라면, 해당 view에 맞는 이벤트를 추가적으로 발행하고 해당 이벤트를 통해 view 데이터를 생성하면 된다.

> In order to avoid exposing DB changes as a contract to other services
> and creating a coupling between them
> the service can consume the CDC topic and produce an “official” API of change events similar to the event stream created in event-sourcing pattern.

## 3. Context 전파가 없는 경우

EDA로의 전환은 개발자가 최종사용자의 요청을 디버깅하거나 추적하는데 어려움이 있다는 것을 내포하고 있다.

![사진1.png](/files/0a710dbd-83f9-1069-8184-b90c37520e08)


1. 기존의 구조들과는 달리, 명확한 HTTP chaining 이 없다.
2. 기존의 코드들과는 달리 Event를 발행하거나 구독하는 코드들이 산개해있다.

각각의 micro service 에서 오류가 발생할 수 있는데, 이를 추적할 수 있는 요소가 필요하다.

### 3-1. Automatic Context Propagation

![화면 캡처 2022-11-27 212636.png](/files/0a710dbd-83f9-1069-8184-b90e3ae00e7b)

발행한 이벤트에 식별할 수 있는 값을 넣어서, 특정 유저의 특정 시점의 요청을 모아볼 수 있다던지 하는 기능들을 만들 수 있다.
이를 통해 이슈가 발생했을 때 쉽게 추적이 가능해진다.

혜택에서는 traceId 라는 개념의 데이터를 넣어주고 있는 것으로 알고 있다.

## 4. 무거운 데이터를 이벤트로 발행하는 경우

large event payload의 조건 (5MB를 넘는 경우..)
지연이 발생하고, 처리량이 낮아지고, memory pressure (out of memory) 가 증가할 수 있다.

### 4-1. 압축해서 보내기

kafka 에서도 기본 제공하는 compression 기능을 활용한다.
5MB가 넘어가면, 50% 압축을 해야 좋은 성능을 유지할 수 있다고 주장하고 있다.

application level 에서 압축하는 것보다 kafka level 에서 압축하는 것이 효율이 좋다고 한다.
=> 여러 메시지를 한 번에 압축할 수 있기 때문

### 4-2. Chunk 로 만들어서 보내기

![화면 캡처 2022-11-27 212627.png](/files/0a710dbd-83f9-1069-8184-b90e3ad70e7a)

Pulsar 라는 메시징 큐 서비스에서는 chunking 이 기본기능이지만, Kafka 에서는 application level 에서 진행해야 한다.

[Kafka Chunking-1](https://medium.com/wix-engineering/chunks-producer-consumer-f97a834df00d)
[Kafka Chunking-2](https://medium.com/workday-engineering/large-message-handling-with-kafka-chunking-vs-external-store-33b0fc4ccf14)

chunk를 기존 payload 로 변환하는 작업이 필요한데, 이 방식에서 위 두 가지 예시의 차이가 존재한다.


### 4-3. 별도의 저장소에 저장하고, reference 만 넘겨주는 것

데이터를 전달하지 않고 별도의 저장소(S3)에 저장한 뒤, reference(URL 등)을 넘겨주면 consumer 가 해당 reference를 통해서 데이터를 받아올 수 있다.
이 방식에서는 실제로 업로드가 잘 되었는지 확인 뒤 메시지가 발행되어야 한다는 점이다. 업로드가 제대로 되지 않았다면, consumer 에서는 계속 재시도를 해야하기 때문이다.

> These object stores allow to persist any required size without impacting first byte latency.

## 5. 중복 이벤트를 처리하지 않는 경우

> Most message brokers and event streaming platforms guarantee at least once delivery 

최소 한 번의 메시지 발행을 보장하기 때문에, 중복으로 메시지가 올 수 있다.
즉, 멱등성을 보장해야한다는 의미

### 5-1. Versioning

![화면 캡처 2022-11-27 212603.png](/files/0a710dbd-83f9-1069-8184-b90e3ae20e7e)


> Optimistic locking technique can serve as inspiration in cases where idempotency of event processing is needed. 

[Optimistic locking](https://vladmihalcea.com/optimistic-vs-pessimistic-locking/)
> Optimistic Locking allows a conflict to occur, but it needs to detect it at write time.

Kafka 에서는 Topic Partition Offset 을 Versioning 용도로 사용할 수 있다. (Unique 함을 보장하므로)

> Specifically With Kafka, there is a possibility to configure exactly once semantics
> but still DB duplicate updates can happen due to some failure
> Luckily the txnId in this case can just be the topic-partition-offset which is guaranteed to be unique.