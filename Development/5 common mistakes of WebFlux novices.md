# 5 common mistakes of WebFlux novices

https://medium.com/@nikeshshetty/5-common-mistakes-of-webflux-novices-f8eda0cd6291

https://gatheca-george.medium.com/5-spring-webflux-tips-tricks-common-mistakes-123cb1bbf2fe

webflux 초보자들이 자주하는 5가지 실수에 대해서 이야기하는 글입니다.
여러가지 글을 봐도 다 비슷비슷하네요.

## 1. Difference between map and flatMap

- flatMap
    - non-blocking 로직 / mono, flux를 반환하는 로직에 사용되어야 함.
- map
    - object/data 를 정해진 시간(fixed time) 에 변환하는 작업을 수행해야할 때
    - 동기적으로 수행이 필요할 때

### 1.1 example

~~~kotlin
return Mono.just(Person("name", "age:12"))
    .map { person -> EnhancedPerson(person, "id-set", "savedInDb") }
    .flatMap { person -> 
        reactiveMongoDb.save(person)
    }
~~~

EnhancedPerson 으로 변환하는 과정은 synchronous/deterministic
reactiveMongoDb.save 과정은 asynchronous/non-deterministic

## 2. Using nested flatMap / map

중첩된 map,flatMap 을 사용하면 읽기가 힘들어진다.
"Do One Thing and Do It Well"

### 2.1 example

#### Bad Practice

~~~kotlin
fun makePersonASalariedEmployee(personId: String): Mono<Person> {
    return personRepository.findPerson(personId)
        .flatMap { person -> 
            employeeSerivce.toEmployee(person)
                .flatMap { employee ->
                    salariedEmployeeService.toSalariedEmployee(employee)
                }
        }
}
~~~

#### Good Practice

~~~kotlin
fun makePersonASalariedEmployee(personId: String): Mono<Person> {
    return personRepository.findPerson(personId)
        .flatMap { person -> 
            employeeSerivce.toEmployee(person)
        }
        .flatMap { employee ->
            salariedEmployeeService.toSalariedEmployee(employee)
        }
}
~~~

## 3. Utilizing reactor library functions like filter, switchIfEmpty, onErrorReturn

built-in 기능들을 잘 활용하도록 하자.

### 3.1 example

#### Bad Practice
~~~kotlin
return Mono.just(Person("name", false))
    .flatMap { person ->
        if (person.consentToSave)
            reactiveMongoDb.save(person)
        Mono.empty()
    }
~~~

#### Good Practice
~~~kotlin
return Mono.just(Person("name", false))
    .filter { person -> person.consentToSave }
    .flatMap { person -> reactiveMongoDb.save(person) }
~~~

## 4. Using Reactive Components as far as possible

Reactive Programming 의 장점은 모든 Component를 비동기적으로 관리할 수 있다는 점인데, 이 장점을 가능한 활용할 수 있도록 return type을 Mono, Flux로 처리하자.

### 4.1 example

#### Bad Practice

~~~kotlin
Flux.range(1, 10)
    .map { doSomeOperation(it) }

fun doSomeOperation(number: Int): Int {
    //
}
~~~

#### Good Practice

~~~kotlin
Flux.range(1, 10)
    .flaMap { doSomeOperation(it) }

fun doSomeOperation(number: Int): Mono<Int> {
    //
}
~~~