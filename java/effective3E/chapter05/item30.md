# 💎 5장: 제네릭

이번 장은 제네릭의 이점을 살리고 단점을 최소화 하는 방법으로 제네릭을 효율적으로 사용하는 방법에 대해 다룬다.

| 한글 용어 | 영문 용어 | 예 | 아이템 |
| --- | --- | --- | --- |
| 매개변수화 타입 | parameterized type | `List<String>` | 아이템 26 |
| 실제 타입 매개변수 | actual type parameter | `String` | 아이템 26 |
| 제네릭 타입 | generic type | `List<E>` | 아이템 26, 29 |
| 정규 타입 매개변수 | formal type parameter | `E` | 아이템 26 |
| 비한정적 와일드카드 타입 | unbounded wildcard type | `List<?>` | 아이템 26 |
| 로 타입 | raw type | `List` | 아이템 26 |
| 한정적 타입 매개변수 | bounded type parameter | `<E extends Number>` | 아이템 29 |
| 제네릭 메서드 | generic method | `static <E> List<E> asList(E[] a)` | 아이템 30 |
| 타입 토큰 | type token | `String.class` | 아이템 33 |

## ✨ 아이템 30: 이왕이면 제네릭 메서드로 만들라

타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입이 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다. 하지만 이렇게 하려면 요청 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴을 제네릭 싱글턴 팩터리라 한다.

## 제네릭 싱글턴 팩토리 패턴

항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이다. `T` 가 어떤 타입이든 `UnaryOperator<T>` 를 사용해도 안전하다.

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```

## 재귀적 타입 한정

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비어있습니다.");
    }
    E result = null;
    for (E e : c) {
        if ( result == null || e.compareTo(result) > 0 ) {
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```

타입 한정인 `<E extends Comparable<E>>` 는 모든 타입 E 는 자신과 비교할 수 있다. 재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴하지만, 다행히 그런 일은 잘 일어나지 안흔ㄴ다.

## 정리

클라리언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다.
