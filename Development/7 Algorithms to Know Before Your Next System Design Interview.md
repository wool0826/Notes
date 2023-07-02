## 개요

https://medium.com/gitconnected/7-algorithms-to-know-before-your-next-system-design-interview-d1de2f374ffa

분산 시스템에서 발생할 수 있는 문제를 해결할 수 있는 기술들에 대해서 설명된 글입니다.

1. Merkel Tree
2. Consistent Hashing
3. Read Repair
4. Gossip Protocol
5. Bloom Filter
6. Heartbeat
7. CAP and PACELC Theorems

## 1. Merkel Tree

![image-20230108-214457-669.png](/files/0a710ba9-8571-16d2-8185-9169a0244458?1673182123158)
서버 간의 데이터 불일치를 식별하는데 사용합니다.

분산 시스템은 fault tolerance, High Availabilty를 위해 각각의 서버에 데이터의 복제본을 관리해야하는데
각 시스템의 데이터가 일치하는데 확인하기 위한 효율적인 알고리즘이 필요합니다.

데이터가 너무 많기 때문에, 단순히 범위를 설정해서 체크섬을 계산하는 방식은 비현실적입니다.

> A replica can contain a lot of data. Naively splitting up the entire range to calculate checksums for comparison is not very feasible. there is simply too much data to be transferred. 

> Instead, we can use Merkle trees to compare replicas of a range.

Merkel Tree는 Hash 이진트리로, 각 노드에서는 하위 노드의 해시값을 계산하여 저장하고 있고 leaf node 에서는 특정 데이터의 해쉬값을 저장하고 있습니다.

중간 -> 왼쪽 -> 오른쪽 (preorder traversal) 순서로 노드 값이 일치하는지 확인합니다.

### 특징

장점

1. 데이터를 모두 전달하지 않기 때문에, 네트워킹으로 인한 비용이 적어서 데이터 동기화의 비용이 적습니다.
2. 비교를 위해 전체 트리가 필요하지 않고, 각 서브 트리들을 독립적으로 확인할 수 있습니다.
    - The principal advantage of a Merkle tree is that each branch of the tree can be checked independently without requiring nodes to download the entire tree or the entire data set.
3. 트리를 순회하면서 처리하므로, 어떤 부분의 파일이 문제가 있는지 쉽게 파악할 수 있습니다.

단점

1. 특정 부분의 파일이 변경되면 트리를 전체적으로 재계산해줘야 합니다.
> 계산의 비용이 그렇게 크지 않을 것 같아서, 그리 큰 단점인지는 모르겠습니다.


### 사용처

Amazon의 DynamoDB

## 2. Consistent Hashing

> https://medium.com/interviewnoodle/how-to-use-consistent-hashing-in-a-system-design-interview-b738be3a1ae3

데이터를 서버 간 분산시켜 저장하기 위해 Consistent Hashing 알고리즘을 사용합니다.
일종의 샤딩키를 계산/관리하는 방식을 의미하는 것 같습니다.

![스크린샷_20230108_094534.png](/files/0a710ba9-8571-16d2-8185-916ac82b445d?1673182133556)![스크린샷_20230108_094547.png](/files/0a710ba9-8571-16d2-8185-916b2efe4465?1673182150878)
단순 해싱값을 N으로 나눈 나머지 값으로 샤드를 선택하면 안 되는 걸까?
=> 데이터 저장/조회에는 문제없이 동작은 하겠지만, 새로운 서버가 추가/제거된다면? 모든 key 값과 물리서버의 맵핑을 다시 해야합니다.
=> 서버의 추가/제거때마다 이 작업을 최소한으로 수행할 수 있도록 합니다.

![스크린샷_20230108_085142.png](/files/0a710ba9-8571-16d2-8185-916aecc84461?1673182142796)
그렇다면 복제는 어떻게 할 것인가? 에 대한 의문이 들 수 있습니다.
복제의 경우, 노드에서 시계방향으로 있는 N개의 노드에 복제를 진행하게 됩니다.

> Each key is assigned to a coordinator node (generally the first node that falls in the hash range), which first stores the data locally and then replicates it to N-1 clockwise successor nodes on the ring. 

### 특징

- 분산시스템에서 물리서버에 데이터를 저장할 때 Mapping 데이터를 통해, 어떤 서버에 어떤 데이터가 저장되어있는지 접근이 가능합니다.
- 서버가 추가/제거될 때 remapping 과정을 최소화합니다.

### 사용처

Amazon의 DynamoDB
Apache의 Cassandra

## 3. Read Repair

> https://towardsdatascience.com/how-to-fix-cassandra-consistency-issues-using-read-repair-6dd45911819d

특정 데이터를 읽는 시점에 유효하지 않은 데이터를 사용하지 않고 최신 데이터를 조회할 수 있도록 복구해주는 알고리즘입니다.

카산드라의 예시

> https://cassandra.apache.org/doc/latest/cassandra/operating/read_repair.html

요청 시 K개의 노드로부터 응답을 받을 수 있도록 옵션을 넣어서 요청한 경우.

1. 가장 응답속도가 빠른 노드는 Coordinator로 full-data를 응답한다.
2. coordinator는 다른 K-1 개의 노드로 요청을 보내 데이터를 조회하게 한다.
    - 이 때 full-data를 반환하는 것이 아니라, hash된 데이터를 반환한다. (네트워킹 비용 감소)
3. coordinator는 해쉬값을 비교해보고, 동일하다면 클라이언트로 응답을 반환한다. 일치하지 않는다면, 문제가 있다는 것이므로 복구작업을 수행한다.
4. 2번에서 요청한 노드로 full-data 조회를 수행한다.
5. 타임스탬프를 기준으로 최신 데이터를 반영한다. (Read-Repair 한다.)
6. 최신화된 데이터를 반환한다.


### 왜 이걸 쓸까?

Cassandra 에서는 트랜잭션을 지원하지 않기 때문에, 데이터 일관성을 유지하기 힘들다.

=> 즉, Read Repair 기능이 꼭 필요하다.

## 4. Gossip Protocol

> https://en.wikipedia.org/wiki/Gossip_protocol

![image-20230108-214719-532.png](/files/0a710ba9-8571-16d2-8185-916bca46446a?1673182163603)
특정 노드가 다른 모든 노드의 상태를 확인하도록 하는 알고리즘입니다.

1. 매초마다 각 노드들은 랜덤하게 선택된 노드와 통신을 해야한다.
2. 통신을 할 때, 각 노드는 자신이 알고 있는 상태정보를 상대에게 공유한다.
    - 1번 노드는 1,2,4,5 노드가 살아있음을 안다.
    - 3번 노드는 3,6번 노드가 살아있음을 안다.
    - 1번, 3번 노드가 통신을 하게 되면 각각 1,2,3,4,5,6 노드가 살아있음을 확인할 수 있다.

> 개인적으로 궁금한 점은, "살아있다"라는 값의 유효한 시간이 있을텐데 따로 타임아웃을 걸고 진행하는 것일지 궁금하네요.

## 5. Bloom Filter

Bloom Filter 자료구조는 `있을 수 있다` 또는 `없는게 확실하다` 를 빠르게 구분할 수 있는 자료구조입니다.

이 자료구조에서 발생할 수 있는 유일한 오류는 `있다고 했으나 없는 경우(false positive)` 입니다.
데이터의 양이 많아질 수록 이 오류의 빈도는 늘어날 수 있습니다.

![image-20230108-214746-590.png](/files/0a710ba9-8571-16d2-8185-916c33ea446f?1673182174926)

1. K개의 Hash Function과 M 크기의 모든 값이 0인 BitArray를 준비합니다
2. 특정 데이터를 K개의 HashFunction을 적용한 뒤, BitArray에서 해당 위치에 1을 기록합니다.
3. 어떤 데이터가 존재하는지 찾고 싶을 때 K개의 HashFunction을 적용한 뒤, BitArray 에서결과를 확인합니다.
    - BitArray 값이 0이 존재하는 경우, `확실히 없음을 보장할 수 있습니다.`
    - 모든 BitArray 값이 1인 경우, `있을 수도 있음을 확인할 수 있습니다.`

## 6. Heartbeat

각 노드의 Health Status를 broadcast 할 때 사용됩니다.
Health Check 같네요.

![image-20230108-214759-668.png](/files/0a710ba9-8571-16d2-8185-916c66fb4474?1673182181916)
1. coordinator 서버가 존재한다면, 해당 서버로 모든 서버들이 주기적으로 Heartbeat 메세지를 전송합니다.
2. coordinator 서버가 없다면, 4번 Gossip Protocol이나 기타 방법들을 사용해 서로 Heartbeat 메세지를 주고받습니다.
3. 특정 타임아웃 내에 메세지를 전달받지 못했다면, 서버의 상태가 정상이 아닐 수 있으므로 추가적으로 요청을 보내거나 하지 않도록 조치를 취합니다.

## 7. CAP and PACELC Theorems

분산시스템을 구성할 때 가이드가 될 수 있는 요소들을 PACELC 라고 명명하고 설명합니다.

![image-20230108-214813-730.png](/files/0a710ba9-8571-16d2-8185-916c9dea4478?1673182188725)
아래 3가지 요소를 모두 충족하는 분산시스템은 없다고 합니다.

- C (Consistency)
    - 모든 노드는 동시에 모두 동일한 데이터를 가지고 있다.
    - 어떤 노드에서나 읽고 쓸 수 있음을 의미.
- A (Availability)
    - 모든 요청은 응답을 받을 수 있다.
    - 하나 또는 그 이상의 노드가 사용 불가능한 상태여도, 요청에 대한 응답을 받을 수 있음을 의미
- P (Partition Tolerance)
    - 네트워킹 이슈 등으로 인해서 노드 간 통신이 불가능할 때도 요청을 처리할 수 있다.
    - 전체 네트워크가 문제가 발생하지 않는다면 서비스의 지속이 가능한 상태를 의미

그래서 우리는 3가지 요소 중 2가지를 골라야합니다. CA, CP, AP.

하지만 CA의 경우 네트워크 문제가 생기면 Consistency 또는 Avaiability를 포기해야하기 때문에 적절한 옵션은 아니라고 합니다.

=> 즉, CP 또는 AP를 선택하라는 의미
=> 즉 Consistency 나 Availability 둘 중 하나를 선택해야 합니다.

- Consistency: ACID (Atomicity, Consistency, Isolation, Durability) 가 중요하다면 RDBMS
- Availability: BASE (Basically Available, Soft-state, Eventually Consistent) 가 중요하다면 NoSQL

### ELC

Partition 이 있을 수 있다면 Consistency-Availability 를 둘 중 하나를 선택해야하고,
Partition 이 없는 환경이라면 Latency-Consistency 를 둘 중 하나를 선택해야한다.