## 인텔리제이 단축키 추천

https://medium.com/dev-genius/10-java-shortcuts-in-intellij-idea-5ee3a8caa02d

### 1. psvm

**p**ublic **s**tatic **v**oid **m**ain
main을 빠르게 생성합니다.

### 2. sout series

System.out.println() 을 빠르게 생성합니다. 

~~~java
// sourceList.soutv
System.out.println("sourceList = " + sourceList);
~~~

~~~java
public List<ClaimDeliveryFeeCalculateSource> buildCalculateSource(
    ClaimDeliveryFeeBundleRequest request,
    OrderContext orderContext,
    Set<String> calculationTargetProductOrderNoSet,
    Set<String> bundleTargetProductOrderNoSet
) {
    // soutm
    System.out.println("ClaimDeliveryFeeCalculateService.buildCalculateSource");

    // soutp
    System.out.println(
    "request = " + request + ", orderContext = " + orderContext + ", calculationTargetProductOrderNoSet = "
        + calculationTargetProductOrderNoSet + ", bundleTargetProductOrderNoSet = "
        + bundleTargetProductOrderNoSet);
}
~~~

## 3. var series

~~~java
// new String("hello").var
String s = new String("hello");

// new String("hello").varl
var s = new String("hello");
~~~

## 4. for series

~~~java
List<A> list = new ArrayList();

//list.for
for (A a : list) {
}
~~~

<img width="589" alt="스크린샷 2023-02-05 12 04 11" src="https://user-images.githubusercontent.com/19607962/216799623-4c0408e1-5acd-453e-a8ee-40714939d2f0.png">

## 5. conditional series

~~~java
boolean a = true

// a.if
if (a) {
    
}

// a.else
if (!a) {
    
}

String s = null;

// s.null
if (s == null) {
    
}

// s.nn
if (s != null) {
    
}

// s.switch
switch (s) {
    
}

## 6. try series

~~~java
// Files.readLines(new File("sfasdf"), Charset.defaultCharset()).try
try {
    Files.readLines(new File("sfasdf"), Charset.defaultCharset())
} catch (IOException e) {
    throw new RuntimeException(e);
}
~~~

## 7. castvar series

~~~java
int a = 3;

// a.castvar, type 따로 적어줘야 함.
long l = (long)a;
~~~

## 8. field series

~~~java
// "MAX_CLAIM_COUNT".field
private final String max_claim_count = "MAX_CLAIM_COUNT";
~~~

<img width="487" alt="스크린샷 2023-02-05 12 13 57" src="https://user-images.githubusercontent.com/19607962/216799671-068f2de6-e264-4a12-b1c9-24cbd95df9d4.png">
<img width="476" alt="스크린샷 2023-02-05 12 14 02" src="https://user-images.githubusercontent.com/19607962/216799672-eba4b141-66bf-4828-923f-374d567746fd.png">


## 9. optional series 

~~~java
// "not-null values".opt
Optional.of("not-null values");

// null.opt
Optional.ofNullable(null)
~~~

## 10. lambda series

~~~java
// System.out.println("hello?").lambda
() -> System.out.println("hello?")
~~~
