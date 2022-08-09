# JDK

## Java 5

시간 관련 Enum `java.util.concurrent.TimeUnit` 추가

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

## Java 11

// TODO https://www.oracle.com/java/technologies/javase/11-relnote-issues.html

## Java 12

## Java 13

## Java 14

### 개선된 스위치 문법

12 에서 미리보기로 시작하여 14 에서 정식지원함

```java
// before
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}

// after
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

추가로 yield 키워드를 사용가능하다. (참고로 변수명으로 쓸 수 있음)

```java
Day day = Day.WEDNESDAY;
int numLetters = switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        yield 6;
    case TUESDAY:
        System.out.println(7);
        yield 7;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        yield 8;
    case WEDNESDAY:
        System.out.println(9);
        yield 9;
    default:
        throw new IllegalStateException("Invalid day: " + day);
};
System.out.println(numLetters);
```

## Java 15

## Java 16

### intanceof 패턴 매칭

[document](https://openjdk.org/jeps/394)

14 에서 미리보기로 시작하여 16 에 정식지원함.

```java
// before
if (obj instanceof String) {
    String o = (String) obj;
    // ...
}

// after
if (obj instanceof String) {
    // obj 는 String 으로 간주됨
    // ...
}
```

`equals()`

```java
// before
@Override
public boolean equals(Object o) {
    if  (!(o instanceof Other other)) {
        return flase;
    }

    Other other = (Other) o;
    return
        && x == other.x
        && y == other.y;
}

// after
@Override
public boolean equals(Object o) {
    return (o instanceof Other other)
        && x == other.x
        && y == other.y;
}
```

### 레코드 클래스

[document](https://docs.oracle.com/en/java/javase/15/language/records.html)

14 에서 미리보기로 시작하여 16에서 정식지원함.

```java
// before
public final class Rectangle {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    double length() { return this.length; }
    double width()  { return this.width; }

    // Implementation of equals() and hashCode(), which specify
    // that two record objects are equal if they
    // are of the same type and contain equal field values.
    public boolean equals...
    public int hashCode...

    // An implementation of toString() that returns a string
    // representation of all the record class's fields,
    // including their names.
    public String toString() {...}
}

// after
record Rectangle(double length, double width) {
}
```

## Java 17

### 봉인 클래스

- [document](https://openjdk.org/jeps/409)
- [document-2](https://docs.oracle.com/en/java/javase/15/language/sealed-classes-and-interfaces.html)

15 에서 미리보기로 시작하여 17 에 정식지원함.

```java
public sealed class Shape
    permits Circle, Square, Rectangle {
}

public final class Circle extends Shape {
    public float radius;
}

public non-sealed class Square extends Shape {
   public double side;
}

public sealed class Rectangle extends Shape permits FilledRectangle {
    public double length, width;
}

public final class FilledRectangle extends Rectangle {
    public int red, green, blue;
}
```

상속 가능한 클래스 또는 인터페이스를 지정할 수 있다. 알려지지 않은 서브 클래스에 대한 관심사가 아니라 알려진 서브 클래스를 통제하는 것에 목적을 가진다.

// TODO 기능 좀 더 확인 필요해보임

## Java 18
