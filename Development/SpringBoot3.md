## Spring Boot 3 (Spring Framework 6) 변경점 훑어보기

### 1. Java 17 (Kotlin 1.7+) 이 지원됩니다.

1. Record
2. Text Block
3. Switch Expression
4. Pattern Matching
5. Sealed Class

와 같은 기능을 사용할 수 있습니다.

### 2. Java EE 가 Jakarta EE 로 대체됩니다.

바뀌는 건 바뀌는 건데 왜 이렇게 바꾸는 걸까? 에 대한 의문이 있었는데요.
상표권 이슈로 변경되었다고 하는 것으로 보입니다.

> 오라클이 자바EE 프로젝트는 이관했지만 자바 상표권은 여전히 보유하고 있기 때문에 자바 네임스페이스 사용에 제약이 있었습니다. 이러한 이유로 자카르타EE에서는 자바 네임스페이스가 Jakarta로, API 패키지명은 javax.* 에서 Jakarta.* 로 변경되었습니다.

### 3. Spring Data 업데이트

1. CRUD repository interfaces 가 iterator 대신 List를 반환합니다. (!!..)
2. 삭제 시 예외처리방식이 변경됩니다. 
    - (? version property 에 대한 내용을 제대로 이해 못 했습니다. / MVCC 같은 얘기같은데 확실치가 않네요.)
    - deleteById: 삭제되는 entity가 없어도 예외를 던지지 않음.
    - delete: 삭제되는 entity가 없으면서 `version property`를 가지고 있다면 OptimisticLockingException 이 발생하고, 이외에는 발생하지 않음.
3. Joda Time, ThreeTenBackport, RxJava1/2 에 대한 지원을 제거합니다.

### 3. GraalVM 이 공식지원됩니다.

AOT Compile 을 사용하기 때문에, 빠른 부팅과 적은 리소스 사용이 요구되는 클라우드 환경에서 적합할 것으로 보고 있습니다.

### 4. HTTP/RSocket Interface Client 를 제공합니다.

> https://docs.spring.io/spring-framework/docs/6.0.0-RC1/reference/html/integration.html#rest-http-interface

WebFlux 의존성이 있어야 사용할 수 있는 기능이라고 하네요.
내부 Client 로직을 구현하지 않고 Interface 만으로도 HTTP/RSocket 호출할 수 있도록 기능을 제공해주는 것으로 이해했습니다.

~~~java
interface RepositoryService {

    @GetExchange("/repos/{owner}/{repo}")
    Repository getRepository(@PathVariable String owner, @PathVariable String repo);

    // more HTTP exchange methods...
}
~~~

~~~java
WebClient client = WebClient.builder().baseUrl("https://api.github.com/").build();
HttpServiceProxyFactory factory = WebClientAdapter.createHttpServiceProxyFactory(client);
factory.afterPropertiesSet();

RepositoryService service = factory.createClient(RepositoryService.class);

// service.getRepository("chanwool-jo", "note"); 를 호출하면 GET https://api.github.com/repos/chanwool-jo/repo 를 호출
~~~

~~~java
@HttpExchange(url = "/repos/{owner}/{repo}", accept = "application/vnd.github.v3+json")
interface RepositoryService {

    @GetExchange
    Repository getRepository(@PathVariable String owner, @PathVariable String repo);

    // 값을 넣어서 메소드를 호출하면, 아래 API를 호출할 수 있다.
    // PATCH https://api.github/com/repos/chanwool-jo/note?name=12344&description=hello?&homepage=www.naver.com
    @PatchExchange(contentType = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
    void updateRepository(@PathVariable String owner, @PathVariable String repo,
            @RequestParam String name, @RequestParam String description, @RequestParam String homepage);

}
~~~

### 5. Micrometer Observation API가 자동으로 구성되며, Observabiltiy 가 공식지원을 시작합니다.

> https://micrometer.io/docs/observation

로깅/메트릭 수집 등의 작업을 위한 기능을 Spring에서 기본으로 제공합니다. (built-in).
micrometer 의존성 추가는 필요한 것으로 보입니다.

### 6. HTTP API 에러 처리를 위한 RFC 7807 스펙을 지원합니다.

> https://www.rfc-editor.org/rfc/rfc7807

Problem 처리를 위해 Zalando Problem 등을 외부 라이브러리를 사용해서 처리를 했었는데요.
Spring Boot 3 에서는 ProblemDetail 이라는 클래스를 통해 기본적으로 예외를 처리할 수 있게 됩니다.

> A common requirement for REST services is to include details in the body of error responses. The Spring Framework supports the "Problem Details for HTTP APIs" specification, RFC 7807.

해당 ProblemDetail 이 RFC-7807 스펙을 지원하는 것으로 보입니다.

- type
    - URI redirection to the webpage which describes a problem type in human-readable format.
- title
    - a quick human-readable summary of what happend
- status
    - http status
- detail
    - explanation of this particular problem
- instance
    - URI of this particular request

~~~XML
HTTP/1.1 403 Forbidden
Content-Type: application/problem+xml
Content-Language: en

<?xml version="1.0" encoding="UTF-8"?>
<problem xmlns="urn:ietf:rfc:7807">
    <type>https://example.com/probs/out-of-credit</type>
    <title>You do not have enough credit.</title>
    <detail>Your current balance is 30, but that costs 50.</detail>
    <instance>https://example.net/account/12345/msgs/abc</instance>
    <balance>30</balance>
    <accounts>
    <i>https://example.net/account/12345</i>
    <i>https://example.net/account/67890</i>
    </accounts>
</problem>
~~~

### 7. 보안상 이슈로 /api/hello 와 /api/hello/ 가 더 이상 일치하지 않습니다.

> https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M4-Release-Notes#upgrading-from-spring-boot-2x

### 8. Logback 및 Log4j2 날짜 및 시간의 기본값이 ISO-8601 표준을 따릅니다.

> https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0.0-M3-Release-Notes#logging-date-format

yyyy-MM-dd’T’HH:mm:ss.SSSXXX 의 형태를 기본값으로 사용합니다.
LOG_DATEFORMAT_PATTERN 값을 사용하면, 기존의 포맷대로 사용할 수 있습니다. (yyyy-MM-dd HH:mm:ss.SSS)


## 출처

- 글 작성할 때 주로 참고한 링크
    - https://revf.tistory.com/260
- spring framework 6.0 변경점
    - https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-6.x/
- javaEE -> jakarta 내용
    - https://www.samsungsds.com/kr/insights/java_jakarta.html