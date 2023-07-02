## What is the Project Loom?

> The primary goal of Project Loom is to support a high-throughput, ligthweight concurrency model in Java.

- OS 스레드를 직접적으로 사용하는 자바에서는 수백만의 요청을 동시에 효율적으로 처리하는 것은 어렵다. (너무 많이 생성하면 OOM 발생함.)
- 이를 해결하기 위해 비동기 프로그래밍을 하곤 하는데, 이는 작성하기도 어렵고 디버깅하기도 어렵다.

이 이야기를 듣고, 이렇게 생각한 사람이 있던 것입니다..

- 성능을 위해 굳이 비동기 API를 사용하지 않고 마음껏 동기-블록킹 API를 사용하자.
- 그대신에 내부에서는 비동기적으로 동작할 수 있게 하자.

위 요구사항을 만족하게 되면 `읽기도 쉽고, 성능도 좋은` 프로그램을 만들 수 있을 것이고 이를 위해 구현하고 있는 기능이 Virtual Thread 인 것으로 이해했습니다.

현재 Java SE 20 에서 2번째 프리뷰를 진행 중입니다.

### 비동기 프로그래밍의 단점

- 제어흐름을 잃는다.
    - [콜백 헬](https://www.google.com/search?q=%EC%BD%9C%EB%B0%B1+%ED%97%AC&client=safari&rls=en&source=lnms&tbm=isch&sa=X&ved=2ahUKEwiN44S3xfn9AhWMfXAKHaDyAyQQ_AUoAXoECAEQAw&biw=1728&bih=926&dpr=2)
    - 이를 개선한 Future, Promise 등에서도 부가적인 코드가 많이 포함되게 된다고 합니다.
- 컨텍스트를 잃는다.
    - 스레드를 넘나들면서 로직이 수행되다보니, 해당 로직의 context 가 스택 트레이스에 유지되지 않습니다.
- 전염성
    - 하나의 기능이 비동기로 구현이 되어있다면, 그 기능을 제대로 활용하기 위해서는 호출하는 부분에서부터 비동기적으로 작성해야합니다.
    - ex) Future를 반환하는 메소드를 사용하는 메소드는 Future를 반환해야함.
    - [함수의 색 문제](https://elizarov.medium.com/how-do-you-color-your-functions-a6bb423d936d)

### 자바 스레드의 특징

1. JVM 스레드와 OS 스레드는 1:1로 매핑된다. OS 스레드 갯수는 CPU 코어 숫자에 의해 제약받는다.
2. OS는 범용적인 스케줄링을 사용한다. 즉, JVM 내부에 대해 알지 못하므로 최적화되어있지 못하다.
3. 스레드의 Context Switching 비용은 비싸다.
4. OS Continuation 구현체는 Java call stack 뿐만 아니라 Native call stack도 포함하며 자원을 많이 사용한다.
    - 이 이유로 인해 스레드를 매 번 생성하는 것이 아니라, Pool 에 넣어놓고 필요 시에 꺼내다 쓰는 식으로 구현이 되어있다.

### 경량 스레드의 장점

자꾸 Carrier Thread 라는 단어가 나오는데, Virtual Thread를 핸들링하고 있는 OS Thread와 1:1 대응되는 Platform Thread를 의미하는 것으로 이해했습니다.

1. JVM에 의해 관리되므로, 스레드 할당 시 System Call 이 필요하지 않다.
2. Context Switch의 영향을 받지 않는다.
    - Carrier Thread 위에서 동작하므로, Context Switch가 필요하지 않다.
3. Virtual Thread가 Blocking 되더라도, Carrier Thread 는 Blocking 되지 않는다.
    - Thread 를 좀 더 효율적으로 사용할 수 있음.
    - 해당 Thread 에서는 다른 작업을 계속 수행하고 있을 수 있음.
4. NIO 나 비동기 API를 사용하지 않아도 된다.
    - 물론 native call 이나 blocking 호출이 필요한 경우에는 Carrier thread 가 blocking 될 수는 있다.
5. 기존 스레드에 비해 메모리 할당량이 적어서, 더 많은 가상스레드를 생성할 수 있다. (수백만개까지 생성 가능)

![java-virtual-threads-1.png](/files/0a710dbd-8630-1145-8187-1dcf1fa542fe)

### Continuation & Scheduler

스케쥴러는 ForkJoinPool을 그대로 사용한다고 합니다.
Continuation 은 low-level API 라 실제 개발자가 사용할 수 있는 API 는 아닙니다.

> 아래 두 문구는 정확히 뭔 소린지 이해는 못 했습니다.

기본적인 OS 스레드는 아래와 같이 스케쥴링됩니다.
`can be executed on the CPU -> parked -> rescheduled back`

Continuations 는 아래와 같이 스케쥴링됩니다.
`can be started -> park (yielded) -> rescheduled back -> resumes it execution`

### VirtualThread 의 상태

PINNED 상태는 독특한데, Blocking Call 을 호출하면 Thread를 점유해야하기 때문에 PINNED 상태로 두고 Platform Thread를 계속 사용하는 상태가 됩니다.

~~~java
// virtual thread state, accessed by VM
private volatile int state;

/*
* Virtual thread state and transitions:
*
*      NEW -> STARTED         // Thread.start
*  STARTED -> TERMINATED      // failed to start
*  STARTED -> RUNNING         // first run
*
*  RUNNING -> PARKING         // Thread attempts to park
*  PARKING -> PARKED          // cont.yield successful, thread is parked
*  PARKING -> PINNED          // cont.yield failed, thread is pinned
*
*   PARKED -> RUNNABLE        // unpark or interrupted
*   PINNED -> RUNNABLE        // unpark or interrupted
*
* RUNNABLE -> RUNNING         // continue execution
*
*  RUNNING -> YIELDING        // Thread.yield
* YIELDING -> RUNNABLE        // yield successful
* YIELDING -> RUNNING         // yield failed
*
*  RUNNING -> TERMINATED      // done
*/
~~~

~~~java
var scope = new ContinuationScope("C1");
var c = new Continuation(scope, () -> {
    System.out.println("Start C1");
    Continuation.yield(scope);
    System.out.println("End C1");
});

while (!c.isDone()) {
    System.out.println("Start run()");
    c.run();
    System.out.println("End run()");
}
~~~

~~~shell
Start run()     ## while 문 시작. c 는 시작되지도 않았으므로, !c.isDone() == true
Start C1        ## c.run() 수행 후 yield 호출하여 대기상태로 진입
End run()       ## while 문 재수행
Start run()     ## while 반복. !c.isDone() == true
End C1          ## c.run() 이 수행됐을 때, 이전에 수행되었던 코드 이후로 수행이 시작됨. (yield 다음부터)
End run()       ## 최종적으로 c.isDone() == true 로 변경되므로 while 도 종료
~~~

위 코드로 Continuation 이 yield 를 수행하더라도, 그 하위의 Carrier thread 는 지속적으로 동작하고 있음을 확인할 수 있습니다.

## How to use it?

아직 preview 라 VM option 에서 --enable-preview 를 켜야 사용할 수 있습니다.

### VirtualThread 직접 만들어서 사용

~~~kotlin
// Factory 를 활용해서 생성
val factory = Thread.ofVirtual().factory()
val virtualThread = factory.newThread { println(Thread.currentThread()) }

// 바로 실행되지 않는 VirtualThread 생성
val virtualThread = Thread
    .ofVirtual()
    .unstarted { var t = "do something" }

virtualThread.start()
virtualThread.join()
~~~

실행결과: VirtualThread[#21]/runnable@ForkJoinPool-1-worker-1


### VirtualThreadPool 사용

`Executors.newVirtualThreadPerTaskExecutor()` 로 VirtualThreadPool 을 쉽게 사용할 수 있습니다.

~~~kotlin
const val MAX_TRANSACTION = 10

fun main(args: Array<String>) {
    val before = Instant.now()
    start()
    val after = Instant.now()

    println(after.toEpochMilli() - before.toEpochMilli())
}

private fun start() {
    Executors.newVirtualThreadPerTaskExecutor().use { virtualThreadExecutor ->
        (1..MAX_TRANSACTION).forEach { index ->
            virtualThreadExecutor.submit() { firstJob(index) }
        }
    }
}

private fun firstJob(num: Int) {
    // Job A,B 2 개를 수행한 후, 두 결과를 취합하여 Job C 를 수행
    val results: List<Future<String>> = Executors.newVirtualThreadPerTaskExecutor().use { virtualThreadExecutor ->
        virtualThreadExecutor.invokeAll(
            listOf(
                Callable<String> {
                    Thread.sleep(1000)
                    return@Callable "$num job A Done."
                },
                Callable<String> {
                    Thread.sleep(300)
                    return@Callable "$num job B Done."
                },
            )
        )
    }

    aggregate(num, results[0].get(), results[1].get())
}

private fun aggregate(num: Int, a: String, b: String) {
    Executors.newVirtualThreadPerTaskExecutor().use { virtualThreadExecutor ->
        virtualThreadExecutor.submit() {
            Thread.sleep(300)
        }
    }
}
~~~

## 별첨-1. Java Release Cycle 

- Preview Features
    - [JEP-12](https://openjdk.org/jeps/12)에 정의되어있습니다.
    - fully specified, fully implemented, and yet impermanent.
    - Allow Java platform developers to communicate whether a new feature is "coming to Java" in approximately its current form within the next 12 months.
    - 버전에 포함하기까지 95% 완료되었다는 느낌
- Experimental Features
    - Experimental features represent early versions of (mostly) VM-level features, which can be risky, incomplete, or even unstable.
    - `which can be risky`
    - 버전에 포함하기까지 25% 완료되었다는 느낌
- Incubating Features
    - Incubating Features are experimental APIs distributed in a form of separate modules with names prefixed with “jdk.incubator.”.
    - incubator 라는 모듈에서 따로 기능을 추가하는 느낌
    - 애매한 기능이면 소리소문없이 사라질 수도 있지 않을까? 싶은 느낌

## 별첨-2. Continuation Passing Style

https://en.wikipedia.org/wiki/Continuation-passing_style

> 약간 보면서 꼬리재귀같은 느낌이 들었던 것 같네요.

## 출처

- http://gunsdevlog.blogspot.com/2020/09/java-project-loom-reactive-streams.html
- https://homoefficio.github.io/2020/12/11/Java-Concurrency-Evolution/
- http://guruma.github.io/posts/2018-09-27-Project-Loom-Fiber-And-Continuation/
- https://www.davidvlijmincx.com/posts/java-virtual-threads/
- https://jenkov.com/tutorials/java-concurrency/java-virtual-threads.html
- https://itnext.io/kotlin-coroutines-vs-java-virtual-threads-a-good-story-but-just-that-91038c7d21eb