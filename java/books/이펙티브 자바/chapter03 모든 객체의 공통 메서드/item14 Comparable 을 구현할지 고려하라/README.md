# ✨ 아이템 14: Comparable 을 구현할지 고려하라

3장에서 다룬 메소드들과 다르게 `Comparable` 의 `compareTo()` 는 `Object` 의 메소드가 아니다. 물론 2가지 성격을 빼면 `equals()` 와 같다.

- `compareTo()` 는 동치성 비교 뿐만 아니라 순서까지 비교한다.
- `compareTo()` 는 제네릭하다.

`Comparable` 을 구현하였다는 것은 해당 클래스의 자연적인 순서가 있다는 것을 뜻한다. 규약은 다음과 같다.

```text
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 **음의 정수**, 같으면 **0**, 크다면 **양의 정수**를 반환한다. 비교할 수 없는 객체 타입이 주어지면 ClassCastException 을 던진다.

모든 x,y 에 대하여 sgn(x.compareTo(y)) == -sgn(y.compareTo(x)) (순서를 보장해야함을 의미)

x.compareTo(y) > 0 && y.compareTo(z) > 0 라면 x.compareTo(z) > 0 이다. (순서의 추이성)

x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 이다. (크기가 같으면 항상 같아야함을 의미)

(x.compareTo(y) == 0) == (x.equals(y)) 여야 하며, 준수하지 못한경우 해당 사실을 명시해야한다.
```

클래스에 핵심 필드가 여러 개라면 어느 것을 비교하느냐가 중요하다. 가장 핵심적인 필드부터 순서대로 비교해나가자. compareTo 메소드에서 필드값을 비교할 때 `<` 와 `>` 연산자를 사용해선 안된다. 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메소드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.

```java
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if ( result == 0 ) {
        result = Short.compare(prefix, pn.prefix);
        if ( result == 0 ) {
            result = Short.compare(lineNum, pn.lineNum);
        }
    }
    return result;
}
```

자바8 에서는 간결하지만 약간 성능의 저하가 있는 깔끔한 구문이있다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber pn) -> pn.areaCode)
                                                                .thenComparingInt(pn -> pn.prefix)
                                                                .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

다른 프리미티브 타입을 커버하며, 객체 참조용 비교자 생성 메서드도 준비되어있다.
