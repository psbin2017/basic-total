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

## ✨ 아이템 27: 비검사 경고를 제거하라

제네릭 사용시 수많은 컴파일러 경고를 보게 된다. 비검사 형변환 경고, 비검사 메소드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등이다.

> javac -Xlint:uncheck

경고를 제거할 수 없지만 타입이 안전하다고 확신한다면 `@SuppressWarnings("unchecked")` 를 달아 경고를 숨기자. 단 검증 없이 경고를 숨기면 런타임에 `ClassCastException` 을 보게 될 것이다. 또한 클래스 범위로 어노테이션을 선언할 수 있지만 가급적 좁은 범위로 선언하는 것이 좋다.

```java
// Before
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        return (T[]) Arrays.copyOf(elements, size, a.getClass());
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length < size) {
        a[size] = null;
    }
    return a;
}

// After
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[] 로 같으므로 올바른 형변환이다.
        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0, a, 0, size);
    if (a.length < size) {
        a[size] = null;
    }
    return a;
}
```

어노테이션 사용시 반드시 경고를 무시해도 안전한 이유를 주석으로 명시하면 좋다. 주석으로 다른사람이 이해하는데 도움이 되며 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다.
