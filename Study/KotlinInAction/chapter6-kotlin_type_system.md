## 6장. 코틀린 타입 시스템

### 6.0. 개요

타입이란?

> 타입은 분류(classification)로, 타입은 어떤 값들이 저장 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정한다
> https://en.wikipedia.org/wiki/Data_type

- Java의 타입시스템은 null을 제대로 다루지 못하고 있다.
    - null 여부를 확인하기 전까지는, 특정 변수에 **어떤 연산을 수행할 수 있을지 알 수 없기 때문**
    - 특정 위치의 변수가 **절대 null일 수 없다고 프로그래머가 확신**하고 검사를 **생략**하는 경우가 있는데, 틀리게되면 **NPE가 발생하며 오류로 중단됨**

코틀린에서는 이러한 자바의 문제점을 해결하고, 가독성을 향상시키기 위해 **Nullable Type**을 제공합니다.

### 6.1.1. 널이 될 수 있는 타입

코틀린에서는 RuntimeException 인 NPE를 컴파일시점에서 검증할 수 있도록 새로운 타입을 추가하였습니다.
타입뒤에 **?** 를 붙임으로써 Nullable 한 타입임을 표시합니다. (이 변수는 널 일수도 있다 **??** 의 느낌?..)

~~~kotlin
fun strLen(s: String) = s.length
fun strLenSafe(s: String?) =
    if (s != null) s.length else 0 // null 체크를 진행하였으므로, 정상 컴파일됨

strLen(null) // 컴파일 시점에서 오류발생
~~~

- Nullable Type 제약조건
    - **변수.메소드()** 와 같이 호출이 불가능
        - null인 경우가 존재하기 때문에 납득가능
        - null 체크를 한 후에는, 변수.메소드() 형태로 호출 가능
    - **Type?** 타입의 값을 **Type** 타입변수에 대입 불가 (파라미터에서도 동일) -> 포함관계이기 때문에 납득가능

### 6.1.3. 안전한 호출 연산자 ?.

호출하려는 값이 null이면 호출은 무시되고 null 결과값이 반환되는 연산자

~~~kotlin
// 동일한 동작을 하는 코드

s?.toUpperCase()

if (s != null) {
    s.toUpperCase()
} else {
    null
}
~~~

~~~kotlin
// chaining이 가능하다

this.company?.address?.country
~~~


### 6.1.4. 엘비스 연산자 ?:

좌항의 값이 null 인 경우, 우항의 값을 결과로 하는 이항 연산자
코틀린에서는 return 이나 throw 등의 연산도 식이므로 우항에 사용할 수 있습니다.

~~~kotlin
// 좌항의 결과가 null인 경우, 기본 값 Unknown을 반환
val address: String = this.company?.address?.country ?: "Unknown"

// 좌항의 결과가 null인 경우, 예외를 throw 하여 메소드를 중단시킬 수 있다.
val address: String = person.company?.address ?: throw IllegalArgumentException("No Address")
~~~

### 6.1.5. 안전한 캐스트 as?

캐스팅이 불가능하다면 null을 반환하고, 가능하다면 캐스팅하여 값을 반환합니다.
일반적인 패턴은 as? 연산자로 캐스팅을 하고, null인 경우를 엘비스 연산자(?:)로 처리하는 것

~~~kotlin
class Person(val firstName: String, val lastName: String) {
    // as?를 사용한 캐스팅 방식
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false
        
        return otherPerson.firstName == firstName
                && otherPerson.lastName == lastName
    }
    
    // 스마트캐스팅을 활용한 처리방식
    fun equalsWithSmartCast(other: Any?): Boolean {
        if (other is Person) { // null 값은 통과하지 못함
            return other.firstName == firstName
                    && other.lastName == lastName
        }

        return false
    }
}
... 
~~~

### 6.1.6. 널 아님 !!

**개발자 판단**하에 널일 수 없는 값에 !! 연산자를 사용하여 **널이 될 수 없는 타입으로 변환**합니다.
!! 연산자는 컴파일러에게 `"나는 이 값이 null이 아님을 잘 알고 있다. 내가 잘못 생각했다면 예외가 발생해도 감수하겠다"` 라고 표현하는 것

!! 연산자를 사용하지 않는 것이 권장되지만, 불가피한 경우도 존재하므로 개발자 판단하에 잘 사용해야합니다.

~~~kotlin
val country = person.company!!.address!!.country // bad practice
~~~

### 6.1.7. let 함수

let 함수를 안전한 호출 연산자(?.) 과 같이 사용하면 null이 될 수 있는 식을 더 쉽게 다룰 수 있습니다.
호출하려는 값이 **null인 경우 무시**되고, **null이 아닌 경우 let 함수를 호출**합니다.

~~~kotlin
fun getEmail(): String? = "chanwool.jo@navercorp.com" // 예시를 위해 nullable Type으로 선언

fun sendEmail(email: String) {
    println("send email to $email")
}

fun exampleOfLetFunction() {
    val email: String? = getEmail()

    // 자바스러운 코드
    if (email != null) {
        sendEmail(email)
    }
    

    // 코틀린스러운 코드
    email?.let { sendEmail(it) }
    
    // 변수로 저장하지 않고도 let 사용이 가능
    getEmail()?.let { sendEmail(it) }
}
~~~

안전한 호출연산자와 같이 사용하지 않는다면 let 내부에서 it은 nullable Type으로 사용됩니다.
[사진 - 1]

### 6.1.8. 나중에 초기화할 프로퍼티

코틀린에서는 널이 될 수 없는 타입은 생성자에서 초기화되어야 합니다.

객체 인스턴스를 생성한 다음에 나중에 초기화하는 프레임워크(DI 프레임워크들)의 경우, lateinit 예약어를 이용하여 
널이 될 수 없는 타입을 **나중에 초기화**할 수 있습니다.

- 나중에 초기화할 프로퍼티 제약조건
    - val 프로퍼티는 final로 컴파일되기 때문에, **항상 var로 선언**하여야 합니다


### 6.1.9. 널이 될 수 있는 타입 확장

일반 타입을 확장하는 것과 유사하게 널이 될 수 있는 타입도 확장이 가능합니다.
**this가 null 일 수 있다는 점** 이 java와의 차이!

~~~kotlin
fun String?.isNullOrBlank(): Boolean =
    this == null || this.isBlank() // 두 번째 this에는 스마트캐스트가 적용됩니다.
~~~

### 6.1.10. 타입 파라미터의 널 가능성

타입 파라미터의 경우, **? 가 붙어있지 않아도 Nullable** 합니다. 이런 예외는 타입 파라미터가 유일!
널이 될 수 없는 타입으로 사용하고 싶은 경우, Upper Bound를 설정해줘야 합니다.

~~~kotlin
fun main() {
    val nullableValue: String? = null

    // 출력결과: null
    nullableValue.myFunction {
        println(nullableValue)
    }

     // compile error!
    nullableValue.myFunctionWithNonnull {
        println(nullableValue)
    }

    // 출력 X
    nullableValue?.myFunctionWithNonnull {
        println(nullableValue)
    }
}

fun <T> T.myFunction(body: (T) -> Unit) {
    body(this)
}

fun <T: Any> T.myFunctionWithNonnull(body: (T) -> Unit) {
    body(this)
}

~~~

### 6.1.11. 널 가능성과 자바

자바의 코드가 코틀린에서 어떻게 해석될 것인지에 대해 설명합니다.

- @NotNull, @Nonnull
    - 널이 될 수 없는 타입으로 변환되어서 사용
- @Nullable
    - 널이 될 수 있는 타입으로 변환되어서 사용
- 어노테이션이 없는 경우
    - 플랫폼 타입으로 처리

#### 플랫폼 타입?

개발자의 책임하에 선택적으로 사용할 수 있는 타입.
**널이 될 수 있는 타입으로 처리해도 되고, 널이 될 수 없는 타입으로 처리해도 됩니다.**.

플랫폼 타입은 null 안정성 검사를 중복수행해도 경고를 표시하지 않습니다.
정말 확실한 경우가 아니라면 null 체크를 하는 것이 안전할 것으로 보입니다.

[사진 - 2]
[사진 - 3]
플랫폼타입인 lastName은 **String!** 으로 표시됩니다. **! 표시는 널 가능성에 대해서 아무 정보도 없다는 의미입니다.**

#### 상속 시 파라미터 타입

~~~java
// java interface

public interface StringUtils {
    boolean isNotBlank(String value);
    boolean isNotBlankWithNotnull(@NotNull String value);
    boolean isNotBlankWithNullable(@Nullable String value);
}
~~~

[사진 - 4]

자바의 인터페이스를 상속하는 경우, 플랫폼 타입으로 인식해서 개발자가 **String?, String 중 하나의 타입을 선택해서 override가 가능**합니다.

### 6.2.1. 원시타입: Int, Boolean / 널이 될 수 있는 원시타입: Int?, Boolean?

코틀린에서는 원시타입과 래퍼타입을 따로 구분하지 않으므로 항상 같은 타입을 사용
**숫자 타입은 컴파일 시에 가능한 한 가장 효율적인 방식으로 변환됩니다**

널이 될 수 없는 타입으로 사용되는 경우, 자바의 primitive type으로 변환이 쉽게 가능

코틀린 타입||자바 타입
:--|:--:|:--
일반적인 경우의 Int, Boolean, ...|→|int, boolean, ...
컬렉션 파라미터 타입으로 사용되는 Int, Boolean, ...|→|Integer, Boolean, ...
Int?, Boolean?, ... 등 널이 될 수 있는 타입|→|Integer, Boolean, ...

### 6.2.3. 숫자변환

코틀린은 **자동으로 형변환을 처리해주지 않습니다!!!..**
to(Type) 함수로 특정 타입으로의 변환을 개발자가 직접 처리해야 합니다.

개발자의 혼란을 피하기 위해 타입변환을 명시하기로 결정됐습니다.
원시타입간의 산술연산의 경우, 연산자 오버로딩을 모두 해둬서 개발자가 따로 신경쓰지 않아도 되게 설정되어있습니다.

[Int 타입의 연산자 오버로드 코드](https://github.com/JetBrains/kotlin/blob/master/core/builtins/native/kotlin/Primitives.kt#L576-L586)

~~~kotlin
val intValue = 10

val longValue: Long = intValue // compile error!
val longValue: Long = intValue.toLong() // ok

val longValue: Long = (10 + 0.1).toLong()
~~~

### 6.2.4. Any, Any?: 최상위 타입

**Java의 Object** 에 해당하는 타입.
코틀린은 원시타입의 최상위 타입도 Any/Any?

Any에는 toString(), equals(), hashCode()는 있지만, wait(), notify()는 없으므로 해당 메소드를 활용하기 위해서는 **Object 로 캐스팅** 해야합니다.

### 6.2.5. Unit: 코틀린의 Void

[사진 - 5]

Unit은 자바 void와 같은 기능을 합니다.

타입 파라미터로 사용했을 때는 java.lang.Void 와 유사한 역할을 하지만, 따로 반환타입이나 return 문을 작성해줄 필요가 없으므로
코드가 더욱 간결해집니다.

**즉, Unit 타입 하나로 void/Void를 모두 커버할 수 있습니다.**

- 특징
    1. Unit 은 싱글톤 인스턴스입니다. 즉, 타입이면서 동시에 객체입니다.
    2. 객체이기도 하기때문에, Any의 서브클래스입니다.
    3. Unit? 는 존재하지 않습니다

~~~kotlin
interface Processor<T> {
    fun process(): T
}

class NoResultProcessor : Processor<Unit> {
    override fun process() {
        // do something..
    }
}
~~~

### 6.2.6. Nothing: 이 함수는 결코 정상적으로 끝나지 않는다.

코틀린 문서에 다음과 같이 설명되고 있습니다.
**Nothing has no instances. You can use Nothing to represent "a value that never exist"**

- 특징
    1. 함수의 실행이 끝나도 호출지점으로 복귀하지 않습니다.
    2. 의도적으로 예외를 발생시킵니다.

void로 선언해서 exception을 던지는 것과 어떤 차이가 있는 지 궁금했는데,
컴파일러에서 최적화를 위해서 사용할 수 있는 것으로 보입니다. (Nothing인 경우 무조건 에러발생이므로)

Nothing? 타입: **예외발생** 또는 **Null**을 반환합니다.

[코틀린의 TODO() 함수](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt#L30)

~~~kotlin
fun main() {
    val nothingExample = NothingExample()
    nothingExample.testTodo() // 여기서 예외발생하면서 더 이상 진행되지 X
    
    // do Something..
}

class NothingExample {
    fun testTodo() {
        TODO("아직 개발 못 한 기능ㅠ")
        println("hello")
    }

    fun nullableNothing(value: String): Nothing? {
        return if (value == "test") {
            throw NotImplementedError("예외 발생!")
        } else {
            null
        }
    }
}
~~~

[사진 - 6]
[사진 - 7]

~~~kotlin
inline fun readUser(id: UserId, onError: (DBError) -> Nothing): User
= ...

fun createUserPage(id: UserId): HtmlPage {
    val user = readUser(id) { err ->
        when (err) {
            is InvalidStatement -> return@createUserPage throw Exception(err.parseError)
            is UserNotFound -> return@createUserPage HtmlPage("No Such User")
        }
    }
}
~~~

위 처럼 error 상황에 특정 처리가 필요한 경우, Nothing 타입을 사용해서 예외상황을 처리할 수 있습니다.

### 6.3.1. 널 가능성과 컬렉션

1. 컬렉션 원소의 널 가능성
2. 컬렉션 자체의 널 가능성

기준|컬렉션 원소가 null|컬렉션 원소가 notnull
:--|:--|:--
컬렉션 자체가 null|`List<Int?>?`|`List<Int>?`
컬렉션 자체가 notnull|`List<Int?>`|`List<Int>`

### 6.3.2. 읽기 전용과 변경 가능한 컬렉션

코틀린을 사용할 때, 일반적으로 "**읽기 전용 인터페이스**" 를 사용합니다.

읽기 전용 컬렉션이라고 해서, 꼭 변경 불가능한 컬렉션일 필요는 없다는 것!
**즉, 읽기 전용 컬렉션이라도 Thread-Safe 하지 않습니다.** 멀티스레드 환경에서는 해당 환경에 맞는 자료구조를 사용해야 합니다.

~~~kotlin
val list: List<Int> = listOf(1,2,3)
(list as MutableList)[1] = 99

println(list) // [1, 99, 3
~~~

### 6.3.3. 코틀린 컬렉션과 자바

- 요약
    1. 코틀린에서 **읽기전용**으로 만들었어도, 자바에서는 **수정이 가능**합니다.
    2. 코틀린에서 **널이 될 수 없는 타입으로 컬렉션**을 만들었어도, 자바에서는 **null을 넣을 수 있습니다.**

### 6.3.4. 자바의 컬렉션을 플랫폼 타입으로 다루기

일반적인 경우는, 어떻게 처리되어도 큰 문제가 없을 가능성이 높습니다. (자바에서 이미 nullable 처럼 처리하고 있으므로)
자바의 컬렉션이 파라미터로 있는 메소드를 코틀린에서 오버라이드하는 경우, 다음의 경우를 고려해야 합니다.

1. 컬렉션이 널이 될 수 있는지?
2. 컬렉션의 원소가 널이 될 수 있는지?
3. 오버라이드하는 메소드가 컬렉션을 변경할 수 있는지?

3가지 모두 처리해야한다면 **MutableList<Int?>?** 와 같은 타입으로 코틀린에서는 처리할 수 있습니다.

### 6.3.5. 객체의 배열과 원시 타입의 배열

~~~kotlin
// java.lang.Integer[] 배열 생성
val nullArray = arrayOfNulls<Int>(10)
val arrayWithInitializer = Array(26) { i -> i * i }
val emptyArray = emptyArray<Int>()

// list -> java.lang.Integer[] 배열 생성
val list = listOf(1, 2, 3, 4)
val arrayFromList = list.toTypedArray()

// int[] 배열
val primitiveTypeArray = intArrayOf(1, 2, 3)

val primitiveTypeArray2 = IntArray(3)
val primitiveTypeArray3 = IntArray(3) { i -> i + 1 }

val primitiveTypeArray4 = boxingTypeArray.toIntArray()
~~~

~~~kotlin
val array = arrayOf("123", 5, 7, 7.0)
~~~