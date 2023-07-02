# 목차

- Java 20 Officially Released. (오늘 할 주제)
    - https://medium.com/@snleepsinipu/java-20-is-officially-released-and-its-amazing-278c6b0809a1
- Spring Boot 3.0 features 
    - https://medium.com/thefreshwrites/spring-3-0-and-its-features-b423374b5dd4
    - https://medium.com/javarevisited/what-are-the-new-features-of-springboot3-6ddba9af664
    - https://medium.com/@nonowak/whats-new-in-spring-boot-3-0-bed0b3ed080f
- Apache Thrift 

## Java 20 Officially Released..

https://medium.com/@snleepsinipu/java-20-is-officially-released-and-its-amazing-278c6b0809a1

jdk20는 6개월간의 지원을 합니다. 다음 LTS 버전은 jdk21 (2023년 9월 예정).
jdk20은 7개의 JEP(all in various incubation and preview stages)를 포함하여 배포되었습니다.

- JEP-429: Scoped Values (Incubator)
- JEP-432: Record Patterns (Second Preview)
- JEP-433: Pattern Matching for switch (Fourth Preview)
- JEP-434: Foreign Function & Memory API (Second Preview)
- JEP-436: Virtual Thread (Second Preview)
- JEP-437: Structured Concurrency (Second Incubator)
- JEP-438: Vector API (Fifth Incubator)

각 기능들에 대해서 간단히 살펴보겠습니다.

### JEP-429: Scoped Values (Incubator)
> https://openjdk.org/jeps/429

ThreadLocal와 유사하게, 특정 값을 저장하고 있는 객체로 쓰레드 내에서는 어디서든 해당 값에 접근해서 사용할 수 있는 것으로 보입니다.
VirtualThread 에서 사용할 ThreadLocal 을 새로 기획한 것으로 이야기하는 것 같네요.

#### ThreadLocal 과의 차이점

- 관리의 용이성
    - where() 에서 설정한 값이 where 메소드 이후에 호출되는 Runnable 에서만 유지됩니다.
    즉, 해당 where() 에서 호출되는 로직이 종료되면, GC에 의해 해제될 수 있습니다.
    - 반면 ThreadLocal 은 스레드가 종료되거나 ThreadLocal.remove() 의 호출을 해야 해당 값이 삭제되고, 개발자는 이를 계속 인식하고 있어야 합니다.
    ThreadPool 에서는 스레드가 종료되지 않기 때문에, 이상한 값을 재사용하는 경우가 발생할 수 있습니다.
- 값을 Immutable 하게 관리할 수 있음.
    - scope 내에서 값을 변경하는 작업은 불가능하고, rebinding 만 가능하기 때문에 Immutable 하게 관리되어 이해하기 쉽습니다.
    - 반면, ThreadLocal 은 mutable 하게 관리될 수 밖에 없는 구조입니다.
- 자식 스레드에도 공유할 수 있음.
    - 자식 스레드에 값을 쉽게 공유할 수 있습니다.
    - 반면, ThreadLocal 에서는 InheritableThreadLocal 을 이용해 값을 공유할 수 있는데, 값을 각각의 스레드에 `복사`해서 처리하는 것이라 리소스가 낭비될 수 잇습니다.

#### Examples

사용방식은 변할 수 있기 때문에, 개념만 확인하시면 될 것 같습니다.

~~~java
class Server {
    public final static ScopedValue<User> LOGGED_IN_USER = ScopedValue.newInstance();

    private void serve(Request request) {
        User loggedInUser = authenticateUser(request);
        ScopedValue
            .where(LOGGED_IN_USER, loggedInUser) // 값 binding
            .run(() -> restAdapter.processRequest(request)); // 전달한 callable 에서만 binding 된 값이 유효함.
    }
}

class RestAdapter {
    public void processRequest(Request request) {
        
        // 전달한 callable 의 호출이 종료되면 LOGGED_IN_USER 는 다시 loggedInUser 로 처리됨.
        UUID id = ScopedValue
            .where(LOGGED_IN_USER, null) // 값 Rebinding
            .call(() -> extractId(request)); // 해당 값을 사용하여 extractId 를 수행.

        useCase.invoke(id);
    }
}

class UseCase {
    public void invoke(UUID id) {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // fork 로 생성되는 자식 스레드에서도 해당 scope 의 값을 확인할 수 있게 처리할 수 있다.
            Future<Data> dataFuture = scope.fork(() -> repository.getData(id));
            Future<ExtData> extDataFuture = scope.fork(() -> remoteService.getExtData(id));

            scope.join();
            scope.throwIfFailed();

            Data data = dataFuture.resultNow();
            ExtData extData = extDataFuture.resultNow(); 
        }
    }
}
~~~

### JEP-432: Record Patterns (Second Preview)
> https://openjdk.org/jeps/432

#### Motivations

instanceof 로 타입이 확인이 된 경우, 추가적으로 캐스팅하지않고 바로 접근해서 사용할 수 있는 기능이 JDK16에 추가되었는데
레코드에서도 동일한 작업이 가능하면 좋겠다.

JDK17, 18, 19를 통해 switch 에서도 해당 패턴매칭 기능이 preview 로 추가되었다.

~~~java
// old java
if (obj instanceof String) {
    String s = (String)obj;
    // use s
}

// new java
if (obj instanceof String s) {
    // use s
}
~~~

#### Examples

~~~java
record Point(int x, int y) {}
enum Color { RED, GREEN, BLUE }
record ColoredPoint(Point p, Color c) {}
record Rectangle(ColoredPoint upperLeft, ColoredPoint lowerRight) {}

static void printSum(Object obj) {
    if (obj instanceof Point p) {
        int x = p.x();
        int y = p.y();
    }
}

static void printSum2(Object obj) {
    // 내부 컴포넌트를 바로 deconstruct 처리할 수 있음.
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x + y);
    }
}

static void printSum3(Object obj) {
    // 필요한 값만 deconstruct 처리할 수 있음.
    if (obj instanceof Rectangle(ColoredPoint(Point p, Color c), ColoredPoint lr)) {
        System.out.println(p.x + p.y);
    }
}

static void printSum3(Object obj) {
    // var 를 사용하여서 타입을 컴파일러가 추론하도록 할 수도 있음.
    if (obj instanceof Rectangle(ColoredPoint(Point(var x, var y), var c), var lr)) {
        System.out.println(x + y);
    }
}

static void forLoop(Rectangle[] r) {
    // for-loop 에서도 deconstruct 를 바로 해서 값에 직접 접근 가능
    for (Rectangle(ColoredPoint(Point p, Color c), ColoredPoint lr) : r) {
        System.out.println(c);
    }
}
~~~

~~~java
record Pair(Object x, Object y) {}

Pair p = new Pair(42, 42);

// 내부 컴포넌트에 대한 패턴매칭도 가능
if (p instanceof Pair(String s, String t)) {
    System.out.println(s + " " + t);
} else {
    System.out.println("Not a pair of Strings");
}
~~~

### JEP-433: Pattern Matching for switch (Fourth Preview)
> https://openjdk.org/jeps/433

#### Examples

~~~java
static String formatterPatternSwitch(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}

static void testFooBar(String s) {
    switch (s) {
        // switch 에서 null 에 대한 처리가 가능해짐.
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}

static void testTriangle(Shape s) {
    switch (s) {
        case null -> 
            { break; }
        case Triangle t
            when t.calculateArea() > 100 -> // Triangle 인데 특정 조건을 만족하는 case만 처리할 수 있게 됨.
                System.out.println("Large triangle");
        case Triangle t ->
            System.out.println("Small triangle");
        default ->
            System.out.println("A shape, possibly a small triangle");
    }
}
~~~

### JEP-434: Foreign Function & Memory API (Second Preview) (aka. FFM)
> https://openjdk.org/jeps/434

java runtime 외부와 통신하기 위해서 API를 제공

> By efficiently invoking foreign functions (i.e., code outside the JVM), and by safely accessing foreign memory (i.e., memory not managed by the JVM), the API enables Java programs to call native libraries and process native data without the brittleness and danger of JNI.

#### Goals

- Ease of Use
    - JNI를 대체하여 순수자바모델을 사용합니다.
- Performance
    - JNI / Unsafe 와 같은 기존 API보다 더 나은 성능을 제공합니다.
- Generallity
    - 다양한 종류의 외부 메모리에서 작동하도록 하며, 시간이 지남에 따라 다른 플랫폼 및 C 외의 다른 언어로 작성된 외부함수를 사용할 수 있도록 합니다.
- Safety
    - 외부 메모리에서 안전하지 않은 작업을 할 수 있지만, 사용자에게 해당 작업에 대해서 경고합니다.

#### Description

FFM API는 아래와 같은 class/interface로 구성되어있습니다.

- Allocate foreign memory
    - MemorySegment
    - SegmentAllocator
- Manipulate and access structured foreign memory
    - MemoryLayout
    - VarHandle
- Control the allocation and deallocation of foreign memory
    - SegmentScope
    - Arena
- Call foreign functions
    - Linker
    - FunctionDescriptor
    - SymbolLookup

#### Examples

~~~java
// 1. 외부 C libary의 기능을 조회합니다. (radixsort(기수정렬) 함수를 가져온 것으로 보임.)
Linker linker          = Linker.nativeLinker();
SymbolLookup stdlib    = linker.defaultLookup();
MethodHandle radixsort = linker.downcallHandle(stdlib.find("radixsort"), ...);

// 2. java heap memory 에 string array 할당
String[] javaStrings = { "mouse", "cat", "dog", "car" };

// 3. java heap 메모리 외부의 메모리에 접근하기 위해 try-with-resource 로 arena 선언
try (Arena offHeap = Arena.openConfined()) {

    // 4. 외부메모리에 2번에서 선언한 대상을 저장할 메모리를 할당함.
    MemorySegment pointers = offHeap.allocateArray(ValueLayout.ADDRESS, javaStrings.length);

    // 5. java heap memory 에서 외부 메모리로 값을 복사
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = offHeap.allocateUtf8String(javaStrings[i]);
        pointers.setAtIndex(ValueLayout.ADDRESS, i, cString);
    }

    // 6. 외부 함수를 호출하여, 외부 메모리의 데이터를 정렬
    radixsort.invoke(pointers, javaStrings.length, MemorySegment.NULL, '\0');

    // 7. 정렬된 데이터를 java heap memory 로 복사
    for (int i = 0; i < javaStrings.length; i++) {
        MemorySegment cString = pointers.getAtIndex(ValueLayout.ADDRESS, i);
        javaStrings[i] = cString.getUtf8String(0);
    }
} // 8. 할당된 외부 메모리는 try를 종료하면서 같이 해제됨.

assert Arrays.equals(javaStrings, new String[] {"car", "cat", "dog", "mouse"});  // true
~~~

### JEP-436: Virtual Thread (Second Preview)
>https://openjdk.org/jeps/436

저번에 설명된 내용이라 넘어가겠습니다.

### JEP-437: Structured Concurrency (Second Incubator)
> https://openjdk.org/jeps/437

JEP-429: Scoped Values (Incubator) 에서 설명한 StructuredTaskScope 를 활용합니다.

#### Unstructured concurrency with ExecutorService

~~~java
Response handle() throws ExecutionException, InterruptedException {
    Future<String>  user  = esvc.submit(() -> findUser());
    Future<Integer> order = esvc.submit(() -> fetchOrder());
    
    String theUser  = user.get();   // Join findUser
    int    theOrder = order.get();  // Join fetchOrder
   
    return new Response(theUser, theOrder);
}
~~~

위 방식에는 크게 3가지 문제가 있는 것으로 분석하고 있습니다.
간단히 설명해서 모두 task-subtask 의 관계의 문제로 설명할 수 있습니다.

1. findUser 가 예외를 발생시키더라도, fetchOrder()는 할당된 스레드 내에서 수행되고 있는 문제
    - 최상의 상황은 thread leak, 최악의 상황은 fetchOrder로 인해 다른 task 에 영향을 주는 상황
2. handle 을 수행하는 thread가 interrupt 되었을 때, subtask로 해당 상태가 전파되지 않으므로, handle()의 수행이 실패하더라도 findUser() / fetchOrder() 는 계속 수행될 수 있는 문제
3. fetchOrder()는 진작에 실패했는데, findUser() 가 수행에 오랜 시간이 걸려서 불필요하게 handle 스레드가 대기하는 문제

제안서에서는 subtask 에서 예외가 발생했을 때, 관련된 모든 task가 해당 상태를 알고 싶어하는 것으로 보입니다.

#### Structured Concurrency with StructuredTaskScope

~~~java
Response handle() throws ExecutionException, InterruptedException {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        Future<String>  user  = scope.fork(() -> findUser());
        Future<Integer> order = scope.fork(() -> fetchOrder());

        scope.join();           // Join both forks
        scope.throwIfFailed();  // ... and propagate errors

        // Here, both forks have succeeded, so compose their results
        return new Response(user.resultNow(), order.resultNow());
    }
}
~~~

- Error handling with short-circuiting
    - ShutdownOnFailure() 를 통해서 findUser() / fetchOrder() 등의 subtask 가 실패했을 때, 다른 subtask도 빠르게 취소처리할 수 있습니다.
- Cancellation propagation
    - handle() 을 수행하는 thread 가 interrupted 되었을 때, scope.fork() 로 수행된 기능들도 자동적으로 취소됩니다.
- Clarity
    - subtask를 정의하고, 해당 subtask를 기다리고, 성공인지 실패인지를 확인하는 코드가 명확하게 구조를 가지고 있습니다.
- Observability
    - Thread dump 을 떴을 때, task의 hierarchy가 명확히 보인다고 합니다.

### JEP-438: Vector API (Fifth Incubator)
> https://openjdk.org/jeps/438

> Java Vector API provides an abstraction layer to data-parallel capabilities of modern CPUs.

최신 CPU의 동시성을 쉽게 사용할 수 있도록 추상화된 클래스 Vector 를 제공해주는 것으로 보입니다.
각 CPU에 맞는 기능을 제공할 수 있어야하기 때문에 Goals 에 "Platform agnostic" 이 존재하네요.

> Note that HotSpot already supports auto-vectorization which can transform scalar operations into vector hardware instructions. However, this approach is quite limited and utilizes only a small set of available vector hardware instructions.

![1*8vkZM2pm68dYveLfvrPbiA.webp](/files/0a710dbd-87a2-1470-8187-acea48884299)

#### Description

- `Vector<E>`
- `VectorSpecies<E>`

#### Examples

~~~java
// old java
void scalarComputation(float[] a, float[] b, float[] c) {
   for (int i = 0; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
   }
}

// new java

// 한 번에 처리할 수 있는 data-width 를 저장해놓고 사용
private static final VectorSpecies<Integer> SPECIES = IntVector.SPECIES_PREFERRED;

void vectorComputation(float[] a, float[] b, float[] c) {
    // 한 번에 처리할 수 있는 register 의 개수를 정함.
    int upperBound = SPECIES.loopBound(a.length);
    
    int i = 0;

    // 각 register 에서 동시에 처리할 작업을 처리
    for (; i < upperBound; i += SPECIES.length()) {
        var va = FloatVector.fromArray(SPECIES, a, i);
        var vb = FloatVector.fromArray(SPECIES, b, i);
        var vc = va.mul(va)
                   .add(vb.mul(vb))
                   .neg();

        vc.intoArray(c, i);
    }

    // vector 로 한 번에 처리할 수 없는 나머지 작업들은 기존처럼 처리
    for (; i < a.length; i++) {
        c[i] = (a[i] * a[i] + b[i] * b[i]) * -1.0f;
    }
}
~~~