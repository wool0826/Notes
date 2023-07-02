출처: https://blog.devgenius.io/how-to-write-testable-code-why-it-matters-4ae556a940f9

UnitTest는 많으면 많을 수록, 내가 기대한대로 코드가 작동함을 보장할 수 있으나
시간의 제약이 있으므로, 무한대로 코드를 생성할 수는 없습니다.

모든 코드에 대해 테스트코드를 작성하는 것은 불가능하므로
이러한 측면에서 좋은 품질의 코드를 작성하기 위해서 TDD 만한 방법은 없을 것이라고 설명하고 있습니다.

### The TDD intro

test를 코드보다 먼저 작성하는 방식입니다.
> you think about your code first, get fast feedback, and change when necessary

- Scenario / Large Test
    - 전체 어플리케이션에 대해서 테스트. end-to-end 테스트 
    - 느리고, 깨지기 쉬운 특성이 있다.
- Functional / Medium Test
    - 어플리케이션의 컴포넌트마다 격리하여 테스트하는 방식
- Unit / Small Test
    - 어플리케이션의 논리에 집중하는 테스트
    - I/O가 없고 빠르다

### TDD Cycle: Red-Green-Refactoring

https://www.codecademy.com/article/tdd-red-green-refactor

- red: 어떤 걸 개발해야하는지 생각하는 단계
- green: 테스트를 어떻게 통과시킬 수 있을지 생각하는 단계
- refactor: 현재 존재하는 코드를 어떻게 개선시킬지 고민하는 단계

#### 1. Red

사이클의 시작지점
특정 기능의 요구사항을 나타내는 테스트를 작성합니다.

sortArray 라는 메소드를 구현하고 싶고, 해당 메소드는 오름차순으로 정렬해서 결과를 반환하는 요구사항이 있다고 가정합니다.

TDD에서는 Red Phase에서 호출의 결과가 정렬되었음을 확인하는 테스트를 작성합니다.

~~~kotlin
@Test
fun sortArray() {
    // given
    val input = intArrayOf(1, 2, 3, 5, 3)

    // when
    val actual = sortArray(input)

    // then
    for ((i, value) in input.withIndex()) {
        if (i == 0) {
            continue;
        }

        then(value).isGreaterThanOrEqualTo(input[i - 1])
    }
}
~~~

하지만 현재 해당 기능은 구현이 되지 않았기 때문에, 테스트는 무조건 실패합니다.

#### 2. Green

이제 테스트를 작성했으니, 해당 테스트를 어떻게 통과시킬지를 고민합니다.
요구사항을 만족하기 위한 코드를 작성하게 되므로, 정렬된 배열을 반환하는 기능을 구현하게 될 것입니다.

~~~kotlin
fun sortArray(array: IntArray): IntArray {
    // bubble sort 를 구현하였다고 가정..

    return bubbleSortedArray
}
~~~

우선 "동작하기는 하는 코드"를 만들어냈습니다.

#### 3. Refactor

Green Phase 까지 동작하기는 하는 코드를 만들었는데요.
이를 더 효율적으로 동작할 수 있도록 개선하는 작업을 수행해야합니다.

[좋은 테스트의 조건](https://www.codecademy.com/article/tdd-u1-good-test)을 고려하면서 작성해봅니다.

위 예시에서는 버블소트대신 퀵소트를 쓴다던지해서 더 빠르게 동작하게 할 수 있습니다.

#### 4. Refactor 그 이후

Refactor를 완료했다고, 개발이 완료된 것은 아닙니다.
"다시 Red phase로 돌아가지 않기 위해" 지속적으로 노력해야하는데요.

1. 신뢰할 수 있는 피드백을 제공해주고 있는지?
2. 테스트가 잘 격리되어있어서 동시에 수행되어도 문제없는지?
3. production code나 test code 에서 중복을 제거할 수 없는지?
4. production code나 test code 에서 요구사항을 더 잘 표현할 수는 없을지?
5. 더 효율적으로 구현할 수는 없을지?

이러한 질문에 대답하는 과정을 진행하다보면 견고한 프로그램을 만들 수 있을 것이라고 하네요.

### Steps to succeed

#### Building Software like an accountant

실제 금액의 입/출이 있을 때 장부에 기록하는 작업처럼
코드를 작성하게 되면, 해당 코드를 테스트하는 코드를 바로 작성하는 습관을 들이는 것을 권하고 있습니다.

#### Fail First / Fail Fast

guard clause 기법을 이용하여 가능한 빠르게 실패를 반환해야
디버깅할 때 훨씬 유리하다고 하네요.

갑자기 드는 생각인데, 클레임 코드 짤 때 예외가 여기저기 퍼져있는데, 이걸 좀 모아둬볼까 싶긴 하네요.

> Debugging is the worst.

#### Single Responsibility Principle / Dependency Injection

간단히, 응집도(cohesion)는 높이고 결합도(coupling)는 낮추기 위해 SRP / DI를 이용하자로 설명할 수 있습니다.

SRP를 통해 응집도를 높여서 특정 모듈에는 특정기능만 하도록 하고
응집도를 높인 각 모듈들을 DI를 통해 de-coupling 을 수행합니다.

#### Violating The law of demeter
https://en.wikipedia.org/wiki/Law_of_Demeter

the law of demeter = principle of least knowlegde
각 모듈들은 다른 모듈과 가능한 접점이 없어야 한다. == 최소한의 의존성을 가지도록 구성한다.

주문처리를 수행할 때, 아래와 같은 기능들이 필요하다고 가정했을 때 각 기능에 대한 Service를 물고 있도록 구성할 수도 있을 것입니다.

1. 구매자에게 알림을 보내고
2. 재고를 줄이는 작업을 하고
3. 재고가 줄어든 상품의 가격을 올리는 작업을 할 수 있다.(..??)

하지만 각 기능을 의존성을 가지고 있는 것이 아니라, 하위 기능들을 묶어서 최소한의 의존성을 가질 수 있게 할 수 있습니다.

1. NotificationService
2. InventoryManagementService
    - 상품의 재고를 관리하거나, 상품의 가격을 관리하는 서비스

예시로 있는 InventoryManagementService 도 그냥 막 합친 게 아니라 관련된 기능을 묶어놓은 것이기 때문에 문제없습니다.


#### Global State

전역변수에 대한 내용입니다
전역변수는 어디서든 접근이 가능하게 되기 때문에, 의존관계를 가지게 됩니다.

이는 병렬적으로 테스트를 수행하기 힘들게 만들기 때문에 가능한 지역적으로 변수를 선언하고 사용하는 걸 권장하고 있습니다.