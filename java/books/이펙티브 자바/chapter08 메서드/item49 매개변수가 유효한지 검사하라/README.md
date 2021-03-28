# ✨ 아이템 49: 매개변수가 유효한지 검사하라

오류는 가능한 한 빨리 잡아야한다. 오류를 발생한 즉시 잡지 못하면 해당 오류를 감지하기 어려워지고, 감지하더라도 오류의 발생 지점을 찾기 어려워진다. 메소드 몸체가 실행되기 전 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.

public 과 protected 메소드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야한다. `@throw` 를 통해 문서화할 수 있고 `IllegalArgumentException`, `IndexOutOfBoundsException`, `NullPointerException` 중 하나가 될 것이다.

```java
/**
 * 항상 음이 아닌 BigInteger 를 반환하다는 점에서 remainder 메소드와 다르다.
 *
 * @param m 계수 (양수이어야 한다.)
 * @return 현재 값 mod m
 * @throw ArthmeticException m 이 0 보다 작거나 같으면 발생한다.
 */
pulbic BigInteger mod(BigInteger m) {
    if ( m.signnum() <= 0) {
        throw new ArthmeticException("계수(m)는 양수여야 합니다." + m);
    }
    // ...
}
```

## `java.util.Objects`

클래스 수준의 주석은 그 클래스의 모든 public 메소드에 적용되므로 각 메소드에 일일히 기술하는 것보다 훨씬 깔끔한 방법이다. 자바 7 에 추가된 `java.util.Objects.requireNonNull` 메소드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다.

```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

자바 9 에서는 Objects 에 범위 검사 기능도 더해졌다. `checkFromIndexSize`, `checkFromToIndex`, `checkIndex` 메소드 들인데 null 검사 메소드 만큼 유연하진 않다. 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계되었다. 또한 닫힌 범위(closed range; 양 끝단 값을 포함하는)는 다루지 못한다. 그래도 이런 제약이 걸림돌이 되지 않는 상황이라면 유용하고 편리하다.

## 단언

```java
public static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    // ... 계산 수행
}
```

- 실패하면 `AssertionError` 를 던진다.
- 런다임에 아무런 효과도, 성등 저하도 없다. (-ea --enableassertion 플래그 설정시 런타임에 영향을 준다.)

메소드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경 써서 검사해야한다.

생성자의 경우로 보면 **생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게함에 있다.**

## 예외

이 규칙에도 예외가 있다.

- 유효성 검사 비용이 지나치게 높을 경우
- 계산 과정에서 암묵적으로 수행되는 경우

예를 들어 `Collections.sort(List)`  처럼 객체 리스트를 정렬하는 메소드이다. 정렬 과정에서 상호 비교될 수 없는 타입의 객체가 있다면 `ClassCastException` 을 던질 것이다. 따라서 모든 객체가 상호 비교될 수 있는지 검사해봐야 실익이 없다.

## 정리

**메소드나 생성자를 작성한다면 그 매개변수들에 어떤 제약이 있을지 생각해야 한다.** 제약들을 문서화하고 메소드 코드 시작부분에 명시적으로 검사해야 한다. 이 습관을 반드시 기르자.
