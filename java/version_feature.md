# JDK

## Java 7

String in switch, Try Catch Resource 등

### Type Inference 타입 추론

```java
// 7 버전 이전
List<String> list = new ArrayList<String>();

// 7 버전 이후
List<String> list = new ArrayList<>();
```

### [Underscores in Numeric Leterals](https://docs.oracle.com/javase/7/docs/technotes/guides/language/underscores-literals.html)

> 숫자 사이에 언더스코어가 가능하다. 소수점과 진수 사이는 안된다.

```java
float pi1 = 3_.1415F;      // 안됨; .의 앞에 위치
float pi2 = 3._1415F;      // 안됨; .의 뒤에 위치
long socialSecurityNumber1 = 999_99_9999_L;   // 안됨; L의 앞에 위치

int x1 = _52;              // 숫자 표현이 아님 (_로 시작하는 것은 변수명이 됨)
int x2 = 5_2;              // 가능
int x3 = 52_;              // 안됨; 숫자의 끝에 위치
int x4 = 5_______2;        // 가능

int x5 = 0_x52;            // 안됨; 16진수를 나타내는 0x사이에는 불가능
int x6 = 0x_52;            // 안됨; 숫자의 시작에 위치
int x7 = 0x5_2;            // 가능 (16진수)
int x8 = 0x52_;            // 안됨; 숫자의 끝에 위치

int x9 = 0_52;             // 가능 (8진수)
int x10 = 05_2;            // 가능 (8진수)
int x11 = 052_;            // 안됨; 숫자의 끝에 위치
```

어떻게 사용해야할까?

> `int a = 123_456_789`; // 단위 구분용도로

- [번역 출처](https://countryxide.tistory.com/52)

## [Java 8](https://www.oracle.com/java/technologies/javase/8-whats-new.html)

> 32비트를 지원하는 마지막 JDK 로 이후에는 서드파티로만 제공

Interface Default Method, Security(TLS 1.2 등)

### Collection

- Stream API 제공
- 키 충돌이 있는 `HashMap` 성능 향상

### Concurrency

- `java.util.concurrent.ConcurrentHashMap`
- `java.util.concurrent.atomic`
- `java.util.concurrent.ForkJoinPool`
- `java.util.concurrent.locks.StampedLock`

### PermGen 제거

`OutOfMemoryError: PermGen Space error` 메모리 누수 중 하나로 메모리에 로딩된 클래스와 클래스로더가 종료될 때 가비지 컬렉션이 수집하지 않아 발생하였다.

Permanent Generation (PermGen) 은 완전하게 제거되어 Metaspace 라는 공간으로 대체됨. PermSize 및 MaxPermSize 인수는 무시되고 위 오류는 발생하지 않게 되었다.

- 클래스 메타 데이터에 대한 대부분의 할당이 기본 메모리에서 할당.
- 클래스 메타 데이터를 설명하는 데 사용된 클래스가 제거.

메타 공간은 시스템의 메모리 양에 의해 제한된다.

- [출처](http://java-latte.blogspot.com/2014/03/metaspace-in-java-8.html)
- [출처2](https://stackoverflow.com/questions/18339707/permgen-elimination-in-jdk-8)

## Java 9

모듈 시스템(Project Jigsaw) 추가

## Java 10

[var 키워드를 이용한 지역변수 선언 및 타입 추론 가능](https://www.youtube.com/watch?v=tjj-XLk4CSA)

## [Java 11](https://www.oracle.com/java/technologies/javase/11-relnote-issues.html)

// TODO