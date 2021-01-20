# Lombok

Lombok 은 자바 컴파일 시점에서 특정 어노테이션으로 해당 코드를 추가할 수 있는 라이브러리이다. 어노테이션 한줄로 POJO 에서 가독성이 떨어지는 코드 덩어리를 한번에 처리하는 등 유용한 라이브러리이다.

Maven 에서도 쉽게 찾아 볼 수 있으며, Lombok 사이트도 존재한다. 실무에서 빠지지 않는 라이브러리이며, 상당히 유용한 친구이다.

본 학습에서는 간단한 어노테이션 사용 예제와 주의 할점에 대해 이야기하고자 한다. 들어가기 앞서 어노테이션 예제는 실무적인 내용이라기 보단, Lombok 에서 feature 로 제공하는 항목에 대해 간단하게 다룬다. (Java 8)

## @NonNull

```java
import lombok.NonNull;

public class NonNullExample extends Something {
    private String name;

    public NonNullExample(@NonNull Person person) {
        super("Hello");
        this.name = person.getName();
    }
}
```

자신의 메소드나 생성자에 Null-Check 문을 삽입할 수 있다. 생성자의 경우 Null-Check 는 명시적으로 `this()` 또는 `super()` 호출 직후에 삽입된다. 만약 호출 이전에 Null-Check 가 작성되어 있다면 Lombok 은 이를 추가 검사하지 않는다.

> validation-api 가 제공하는 `@NotNull` 과 다른 어노테이션이다. 기능상 다른지는 구체적으로 살펴보지 않았다.

## @Cleanup

```java
import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
    public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
            int r = in .read(b);
            if (r == -1) break;
            out.write(b, 0, r);
        }
    }
}
```

지정된 리소스가 실행 범위를 벗어나기 전에 자동으로 정리(`close()`)된다. 예제로 설명을 덧붙이자면 `@Cleanup` 이 선언된 in 과 out 지역 변수는 종료 시점에 `in.close()` 와 `out.close()` 가 호출 된다. 내부적으로 이 호출은 try / finally 구조로 실행된다.

정리하는 객체의 `close()` 메소드가 없다면 지정할 수 있다. `@Cleanup("dispose") ..`

하나 이상의 인수를 사용하는 `close()` 메소드는 이 어노테이션을 사용할 수 없다.

> `@Cleanup` 은 자원을 반환해야하는 지역 변수를 어노테이션으로 선언되었기 때문에 명시적으로 알 수 있다. 다만 이는 Java 7 에서 지원되는 `AutoCloseable` 인터페이스에 해당되는 지역 변수라면 try / catch / resource 스타일이 어울린다고 판단 된다.

## @Getter 와 @Setter

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {
    /**
     * Age of the person. Water is wet.
     * 
     * @param age New value for this person's age. Sky is blue.
     * @return The current value of this person's age. Circles are round.
     */
    @Getter @Setter private int age = 10;

    /**
     * Name of the person.
     * -- SETTER --
     * Changes the name of this person.
     * 
     * @param name The new value.
     */
    @Setter(AccessLevel.PROTECTED) private String name;

    @Override public String toString() {
        return String.format("%s (age: %d)", name, age);
    }
}
```

특정 필드에 `@Getter` 또는 `@Setter` 를 사용한다면 Lombok 은 해당 필드를 자동으로 메소드를 생성해준다. 만약 타입이 `boolean` 이면 `isFoo` 스타일로 지정된다. 생성되는 메소드에 대해 접근 제한자를 부여할 수 있다.

클래스에 선언한다면 해당 클래스 내부 필드에 모든 필드에 메소드가 생성 된다.

## @ToString

```java
import lombok.ToString;

@ToString
public class ToStringExample {
    private static final int STATIC_VAR = 10;
    private String name;
    private Shape shape = new Square(5, 10);
    private String[] tags;
    @ToString.Exclude private int id;

    public String getName() {
        return this.name;
    }

    @ToString(callSuper = true, includeFieldNames = true)
    public static class Square extends Shape {
        private final int width, height;

        public Square(int width, int height) {
            this.width = width;
            this.height = height;
        }
    }
}
```

클래스에 `@ToString` 을 선언하여 Lombok 이 제공하는 `toString()` 메소드를 구현할 수 있다. 일반 객체의 `toString()` 은 객체의 필드 값이 나타나지 않는다.

기본적으로 모든 비 정적 필드가 나타난다. 일부 필드를 건너뛰고자 한다면 `@ToString.Exclude` 또는 사용하고자 하는 필드를 정확하게 지정하고 포함하고자 하는 필드를 표시할 수 있다.

`callSuper` 옵션은 해당 부모 클래스를 포함하여 `toString()` 출력을 한다. 만약 `toString()` 출력에서 선언한 필드 명이 아닌 다른 필드 명을 원한다면 `@ToString.Include("custom field name")` 으로 사용할 수 있다. 더 해 `@ToString.Include(rank = -1)` 로 출력의 우선 순위를 둘 수 있다.

## @EqualsAndHashCode

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class EqualsAndHashCodeExample {
    private transient int transientVar = 10;
    private String name;
    private double score;
    @EqualsAndHashCode.Exclude private Shape shape = new Square(5, 10);
    private String[] tags;
    @EqualsAndHashCode.Exclude private int id;

    public String getName() {
        return this.name;
    }

    @EqualsAndHashCode(callSuper = true)
    public static class Square extends Shape {
        private final int width, height;

        public Square(int width, int height) {
            this.width = width;
            this.height = height;
        }
    }
}
```

클래스의 정의에 Lombok 이 제공하는 `equals(Object other)` 및 `hashCode()` 를 구현해준다. 이 어노테이션은 클래스를 확장하는 클래스에 적용하면 제공하는 기능이 다소 복잡해진다.

일반적으로 부모 클래스는 위 메소드들이 필요한 필드를 정의하지만, 이 코드는 생성되지 않으므로 이러한 클래스에서는 지양해야한다. 물론 `callSuper` 옵션으로 `super.hashCode()` 알고리즘을 포함하고 있어 커버가 가능하다. 확장하지 않은 클래스에 이 옵션을 사용하면 컴파일 에러를 발생시킨다.

- [How to Write an Equality Method in Java](https://www.artima.com/lejava/articles/equality.html)

> `equals(Object other)` 와 `hashCode()` 는 항상 단일 클래스에서는 큰 문제가 없이 보장되지만 상속 관계를 가지는 클래스들에 대해서 자식 클래스가 문제를 발생시킬 여지가 다소 있다. 다른 학습에서는 이를 회피하기 위한 가장 쉬운 방법으로 `@Override` 하지 않는 것을 권고하였다.

## @...Constructor

```java
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample < T > {
    private int x;
    private int y;
    @NonNull private T description;

    @NoArgsConstructor
    public static class NoArgsExample {
        @NonNull private String field;
    }
}
```

생성자와 관련되어 있는 이 어노테이션이다.

`@NoArgsConstructor` 는 매개변수 없는 기본 생성자를 뜻한다. `@NonNull` 필드가 있다면 나중에 이 필드가 조건을 충족하지 못한다면 당연히 문제가 발생한다는 것을 인지해야 한다. `force` 옵션으로 강제로 타입에 대한 초기 값으로 초기화 할 수 있다.

`@RequiredArgsConstructor` 는 특수 처리가 필요한 각 필드에 대해 1개의 매개변수 생성자를 뜻한다. 초기화 되지 않은 모든 `final` 필드는 매개 변수를 가져오며 `@NonNull` 필드의 생성자라면  Null-Check 를 포함한다.

`@AllArgsConstructor` 는 모든 필드에 대해 매개변수를 가지는 생성자를 뜻한다. `@NonNull` 필드의 생성자라면  Null-Check 를 포함한다.

각각의 어노테이션은 `staticName` 옵션으로 항상 개개인의 대체 양식을 허용하며 개인 생성자를 감싸는 추가 정적 팩토리 메소드가 생성된다. 이로 생성된 정적 팩토리 메소드는 제네릭으로 선언하면 추가적으로 필드를 유추한다.

`staticName` 로 지정하면 정적 팩토리 메소드 생성할 수 있기 때문에 생성할 때 `@NonNull` 에대한 감지가 자동으로 이루어 진다. 다만 필드를 선언한 순서로 지정되기 때문에 주의해야할 것이다.

## @Data

```java
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data public class DataExample {
    private final String name;
    @Setter(AccessLevel.PACKAGE) private int age;
    private double score;
    private String[] tags;

    @ToString(includeFieldNames = true)
    @Data(staticConstructor = "of")
    public static class Exercise < T > {
        private final String name;
        private final T value;
    }
}
```

앞에서 언급한 `@Cleanup` 을 제외한 모든 기능을 포함한다. 추가적인 구체적인 기능을 필요로 하면 위 어노테이션의 규약에 따라 선언해주면 된다. `staticConstructor` 옵션을 사용하여 정적 팩토리 메소드로 생성자 선언을 할 수 있으며 마찬가지로 제네릭으로 선언하면 추가적으로 필드를 유추한다.

## @Value

```java
import lombok.AccessLevel;
import lombok.experimental.NonFinal;
import lombok.experimental.Value;
import lombok.experimental.Wither;
import lombok.ToString;

@Value
public class ValueExample {
    String name;
    @Wither(AccessLevel.PACKAGE) @NonFinal int age;
    double score;
    protected String[] tags;

    @ToString(includeFieldNames = true)
    @Value(staticConstructor = "of")
    public static class Exercise < T > {
        String name;
        T value;
    }
}
```

바로 이전에 살펴본 `@Data` 와 동일해 보이지만 `@Value` 로 선언된 클래스는 생성자로 생성된 이후 불변이다. 즉, `@Setter` 로 선언해주어도 무시된다. 명시적으로 `@NonFinal` 하거나 `@PackagePrivate` 주석을 사용하여 기본 값에 의한 최종 및 개별 동작을 재정의 할 수도 있다.

`@Value` 는 val 어노테이션을 사용하기 때문에 `val` 어노테이션을 사용 가능한 상태에서만 `@Value` 어노테이션을 사용할 수 있다.

## @Builder

```java
import lombok.Builder;
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
    @Builder.Default private long created = System.currentTimeMillis();
    private String name;
    private int age;
    @Singular private Set<String> occupations;
}
```

`@Builder` 는 클래스나 생성자 또는 메소드에 배치할 수 있다. 클래스 및 생성자가 가장 일반적인 사용방법이다. `@Builder.Default` 를 사용하여 기본 값을 설정해 줄 수 있다.

```java
val builderClass = BuilderClass.builder()
                                .name("Name")
                                .name(100)
                                .occupation('occupataion')
                                .build();
```

`@Singular` 를 사용하여 `Collection` 에도 요소를 추가할 수 있으며, setter 명은 선언된 필드 명과 동일하다. 이 때, `@Singular` `Collection` 은 불변이다. (builder 로 생성된게 불변이다.) 복수형으로 필드명을 선언하면 명시적으로 단수형으로 setter 명을 바꿔준다. 이때 식별자에 대한 단수형이 모호할 경우 옵션을 주어 선언할 수 있다.

## @SneakyThrows

```java
import lombok.SneakyThrows;

public class SneakyThrowsExample implements Runnable {
    @SneakyThrows(UnsupportedEncodingException.class)
    public String utf8ToString(byte[] bytes) {
        return new String(bytes, "UTF-8");
    }

    @SneakyThrows
    public void run() {
        throw new Throwable();
    }
}
```

메소드 선언부에 사용되는 throws 키워드 대신 어노테이션으로 예외 클래스를 파라미터로 입력 받는다. 이 어노테이션은 예외 처리 원칙을 무시하는 코드로 될 수 있는데, 일반적인 예외 처리가 아닌 경우에 유용하게 사용할 수 있다.

1. `Runnable` 인터페이스는 `run()` 메소드 안에 발생한 예외는 호출한 쓰레드로 전파 되지 않는다. 모든 예외가 `RuntimeException` 으로 처리되기 때문이다. 정상적인 예외 처리가 힘들 때 유용하게 사용할 수 있다.
2. 위 예제처럼 발생할 수 없는 예외에서 사용할 수 있다. UTF-8 은 JVM 에서 항상 사용이 가능하기 때문에 일어날 수 없는 예외이다.

예외에 대한 특별한 처리를 필요로 하지 않는다면, 위의 경우 외에도 유용할 수 있다.

## @Synchronized

```java
import lombok.Synchronized;

public class SynchronizedExample {
    private final Object readLock = new Object();

    @Synchronized
    public static void hello() {
        System.out.println("world");
    }

    @Synchronized
    public int answerToLife() {
        return 42;
    }

    @Synchronized("readLock")
    public void foo() {
        System.out.println("bar");
    }
}
```

메소드의 사용되는 synchronized 키워드 보다 더 향상된 기능을 제공한다. synchronized 는 static 과 instance 단위로 락 을 걸지만 `@Synchronized` 는 파라미터로 입력 받는 `Object` 단위로 락 을 건다. 설명이 다소 추상적이기 때문에 바닐라 자바로 내용을 더한다.

```java
public class SynchronizedExample {
    private static final Object $LOCK = new Object[0];
    private final Object $lock = new Object[0];
    private final Object readLock = new Object();

    public static void hello() {
        synchronized($LOCK) {
            System.out.println("world");
        }
    }

    public int answerToLife() {
        synchronized($lock) {
            return 42;
        }
    }

    public void foo() {
        synchronized(readLock) {
            System.out.println("bar");
        }
    }
}
```

## 사용법 요약

- `@NoArgsConstructor(access = AccessLevel.PROTECTED)` JPA 에서 프록시 객체가 필요하기 때문에 기본 생성자가 반드시 있어야 한다. 이 경우 접근지시자는 protected 로 지정한다.
- `@Data` 는 지양한다. 너무 많은 기능을 제공한다.
- `@Setter` 는 사용하지 않는다. 객체의 변경 포인트를 남용하게 된다.
- `@ToString` 은 무한 참조가 생길 수 있다. `@ToString(of == {""} )` 권장한다.
- 클래스 상단의 `@Builder` 는 지양하고 생성자 위에 `@Builder` 를 사용한다.

## 학습 내용 참고 출처

- [롬복(Lombok)의 어노테이션들 - (EqaulsAndHashCode, XArgsConstructor, Data, Value, Builder)](https://partnerjun.tistory.com/54?category=693285)
- [롬복(Lombok)의 어노테이션들 - (SneakyThrows, Synchronized, Log)](https://partnerjun.tistory.com/55?category=693285)
- [Lombok](http://kwonnam.pe.kr/wiki/java/lombok)

## 학습 내용 심화 과정

Lombok 은 버전업되면서 내재된 사용법에 대한 버그를 개선하고 있는데 본 문제를 가급적 회피 및 지양하기 위한 내용이 담겨져있다. 사용법 요약에 추가 설명이라고 보면 된다.
가장 정리가 잘되어있다고 생각하는 URL 은 마지막 URL 이다. (내가 정리한게 무색할 정도)
스택 오버 플로우 URL 은 상속 구조의 Lombok 어노테이션에서 warn 에 대한 처리이다.
마지막으로 1.8.0 ++ 버전 사용을 지향하자.

- [Lombok 사용상 주의점(Pitfall)](http://kwonnam.pe.kr/wiki/java/lombok/pitfall)
- [실무에서 Lombok 사용법](https://cheese10yun.github.io/lombok/)
- [lombok 사용 시 주의사항](https://stackoverflow.com/questions/38572566/warning-equals-hashcode-on-data-annotation-lombok-with-inheritance)
- [](https://lng1982.tistory.com/300)
- [Spring Boot 1.5.15, 2.0.3 출시, 롬복(lombok) 1.16.22 관련 내용](https://java.ihoney.pe.kr/510)
- [Lombok 사용법과 주의사항](https://hyoj.github.io/blog/java/basic/lombok/#getter-setter)
