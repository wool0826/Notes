https://medium.com/@interviewready/cache-low-level-design-bda0944f5b74

캐시 구현 방식에 대해서 정리된 문서입니다.
여기서는 key-value 캐시에 대해서만 다룬다고 합니다.

읽으면서 정리하다보니 매 번 늦게 파악하게 되는데, 막 엄청난 내용은 없네요..

다음 주에는 https://www.assemblyai.com/blog/how-chatgpt-actually-works/ 요걸 정리해보려고 합니다.
너무 길어서,,,,,,, 우선 다음 주로 넘겼습니다.

# Core requirements for our cache

## Deleting data

TTL에 의해 삭제되거나

캐시 스토리지가 꽉 차서 과거 데이터를 삭제하고 새로운 데이터를 생성해야 하는 경우에는 
[LRU(Least Recently Used), LFU(Least Frequently Used)](https://3catpapa.tistory.com/111) 등의 알고리즘을 이용하여 삭제합니다.

## Writing data

### Write Back Policy

- Cache 에서 최신 데이터를 계속 가지고 있다가, 한 번에 DB에 최신 값을 갱신하는 방식
- DB를 계속 호출하지 않기 때문에 성능 상으로는 효율적입니다.
- 캐시가 갑자기 죽는 상황이 발생하면, 최신 데이터를 잃을 수 있다는 단점이 있습니다.
- 분산환경에서의 캐시는 캐시 간의 일관성을 유지하기 어렵습니다. (A 캐시에는 최신 값, B 캐시에는 예전 값이 있을 수 있음)

### Write Through Policy

- 캐시의 값이 갱신되면, 동기호출을 통해 DB의 값을 바로 갱신하는 방식
- Write-Back Policy 보다는 성능적인 면에서 비효율적입니다.
- 최신 데이터의 손실위험이 없습니다.
- 유저는 항상 최신의 데이터를 조회할 수 있습니다. (Write-Back Policy의 4번항목 대비)

### Read your own write policy

![0_zbs80eKLRoiGhY4z.webp](/files/0a710ba9-8571-16d2-8185-b5e6ece420e1)

1. 현재 유저가 30명인데, 클라이언트가 유저 1명 추가를 요청
2. `DB에서 쓰는 속도가 느려서` 유저 수 갱신이 늦어지고 있는 상황.
3. 클라이언트가 총 유저 수 조회를 요청 
4. 아직 DB가 갱신되지 못 하여, 최신화되지 못한 30명을 반환

여기서 `클라이언트는 항상 최신화된 데이터를 읽는다` 는 조건을 보장하기 위한 방법은 뭐가 있을지? 에 대해서 설명합니다.

### 사용할 수 있는 기법들?

- Request Collapsing
    - 동일한 key에 대한 요청을 묶어서 캐시로 한 번 요청한 뒤, 응답값을 모든 클라이언트에게 반환할 수 있습니다.
- Hot Loading (Pre-fetching)
    - 데이터가 조회되기 이전에 미리 데이터를 가져오도록 할 수 있습니다.
    (pre-fetch the data and store it in the cache before they are needed or there is a read miss.)
- Event Log
    - 새로운 쓰기, 갱신, 삭제 작업이 발생할 때, DB나 분석엔진으로 로그를 보내서 처리할 수 있습니다.
- Asynchronous processing
    - 비동기 처리를 통해서 클라이언트는 non-blocking 으로 캐시를 조회할 수 있고, 캐시 성능을 향상시킬 수 있습니다.

## Implementing Asynchronous processing

- Blocking Call 은 스레드 수 만큼 호출할 수 있고 스레드의 수는 한정되어있으므로, 많은 요청 & 느린 처리속도가 합쳐지면 요청을 처리하지 못하는 상황도 발생할 수 있습니다.
- [Head Of Line Blocking](https://en.m.wikipedia.org/wiki/Head-of-line_blocking) 이 발생하는 경우, 새로운 요청을 처리할 수 있는 소켓이 줄어들게 되므로, 응답시간이 더 길어지게 된다고 합니다.
    - [HOL Blocking 참고 링크](https://letitkang.tistory.com/79)

이런 이슈를 해소하기 위해 비동기처리를 고민하게 되었다고 합니다.

## Implementing Asynchronous processing with Future Object

Future 라는 클래스를 통해 처리할 것이라고 합니다.
Future는 다들 아실 것 같아 바로 넘어가겠습니다.

## Implementing Read your Own Write Policy

- Using Locks
    - DB 처럼 락을 사용하여, 쓰기처리가 완료된 데이터만 읽을 수 있도록 강제할 수 있습니다.
    - 락을 건 클라이언트가 죽게된다면, 다른 클라이언트는 해당 데이터에 접근할 수 없게 됩니다.
        - 락에 TTL 을 걸거나, 클라이언트가 죽기 전에는 무조건 락을 해제하도록 하여 이러한 문제는 해결할 수 있습니다.
- Ordering Using threads assignment
    - 스레드는 할당된 순서대로 작업을 수행합니다.
        - ex) Thread 1 에 A,B 순서로 작업을 할당하였다면, Thread 1은 A를 수행한 뒤 B를 수행합니다.
    - 특정 키에 대한 작업은 특정 스레드에만 할당하도록 구성합니다. 
    - 이런 특징을 통해 비동기처리를 하지만, 읽고 쓰는 작업에서 일관성을 유지할 수 있습니다.

![0_pAQU1j42Qv6EU3Z9.webp](/files/0a710ba9-8571-16d2-8185-b5e9e0172127)

### 싱글코어에서 예시

![0_Gta-3GzZpBeOXsqO.webp](/files/0a710ba9-8571-16d2-8185-b5e9e0172128)

같은 Key 값에 대한 스레드가 시간을 할당받아 처리하게 되는데, 스레드는 할당된 작업을 순서대로 처리하므로 문제없습니다.

### 멀티코어에서 예시

![0_VZktyAu_m51FdNpH.webp](/files/0a710ba9-8571-16d2-8185-b5e9e0182129)

멀티코어에서도 싱글코어와 동일하게 순서대로 처리하게 되므로 문제없습니다.