# 💎 6장: 열거 타입과 애너테이션

자바에는 특수한 목적의 2 가지의 참조타입이 존재한다. 클래스의 일종인 열거 타입(Enum Type)과 인터페이스의 일종인 애너테이션(Annontation)이다. 이 타입들을 올바르게 사용하는 방법을 학습한다.

## ✨ 아이템 34: int 상수 대신 열거 타입을 사용하라

정수 열거 패턴에는 단점이 많다 타입 안전을 보장할 수 없으며 표현력도 좋지 않다 평범한 상수를 나열한 것 뿐이라 컴파일시 그 값이 클라이언트 파일에 새겨진다. 변경되어 다시 컴파일하지 않는 경우 엉뚱하게 동작할 것이다.

다행히 자바는 열거 패턴의 단점을 말끔히 씻어주는 동시에 여러 장점을 안겨주는 대안이 있다. 열거 타입이다.

- 열거 타입은 컴파일 타임 타입 안전성을 보장한다.
- 열거 타입은 각자의 이름 공간이 있어 이름이 같은 상수도 평화롭게 공존한다.

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6);
    // ...

    private final double mass;
    private final double radius;
    private final double surfaceGravity;

    private static final double GRAVITY = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = GRAVITY * mass (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }

    public dobule surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드로 저장하면 된다.

열거 타입은 근본적으로 불변이라 모든 필드는 final 이어야 한다. 필드를 public 으로 선언해도 되지만 private 로 선언하고 public 접근자 메소드를 만드는 것이 낫다.

```java
for (Planet p : Planet.values()) {
    System.out.printf("%s에서 무게는 %f이다. \n" p, p.surfaceWeight(mass));
}
```

열거 타입은 자신안에 정의된 상수를 값에 배열에 담아 반환하는 values 메소드를 제공한다. **값은 선언 순서로 저장된다.**

만약 특정 상수를 제거한다면 사용하는 곳은 컴파일 에러가 발생하며 이는 정수 열거 패턴에서 기대하지 못한 바람직한 대처를 기대할수 있다. (코드에서 발생하는 명시적으로 에러를 확인하고 수정할 수 있다는 뜻)

## 상수별 특정 동작

[상수별 클래스 몸체](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/ConstantSpecificBody.java)

각 상수별 클래스 몸체(constant-specific class body), 즉 각 상수에서 자신에 맞게 재정의하는 방법이다. 이를 상수별 메서드 구현(constant-specific method implementation)이라고 한다.

## fromString()

```java
private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(Collectors.toMap(Object::toString, e -> e));

public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable( stringToEnum.get(symbol) );
}
```

`stringToEnum` 이 추가되는 시점은 열거 타입 상수 생성후 정적 필드가 초기화 될 때이다. 한편, 상수별 메소드 구현에는 열거 타입 상수끼리 공유하기 어렵다는 단점이 있다.

예를 들어 주말 상수에는 주말 수당이 적용되고 주중은 주중 수당이 적용된다고 가정하면 switch 문을 사용하는 방법 외에 다음 방법이 있다.

## 전략 열거 패턴

[전략 열거 패턴](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/PayRollDay.java)

switch 문을 사용하는 방법보다 다소 코드가 복잡해졌지만, 더 안전하고 유연한 방법이다.

그래서 열거 타입은 언제 사용해야할까?

> 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.

- 태양계 행성.
- 한 주의 요일.
- 체스 말.

그리고 컴파일타임에 이미 알고 있을 때도 쓸 수 있다. 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요도 없다. (제거되도 추가되도 문제없다.)
