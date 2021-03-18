# ✨ 아이템 44: 표준 함수형 인터페이스를 사용하라

필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라. API 가 다루는 개념의 수가 줄어 익히기 쉬워지고, 다른 코드와의 상호운용성도 크게 좋아진다.

`java.util.function` 패키지에는 43 개의 인터페이스가 담겨 있다. 전부 기억하긴 어렵지만 기본 6 개 인터페이스를 기억하면 유추할 수 있을 것이다.

| 인터페이스 | 함수 시그니처 | 예 |
| --- | --- | --- |
| `UnaryOperator<T>` | T apply(T t) | String::toLowerCase |
| `BinaryOperator<T>` | T apply(T t1, T t2) | BigInteger::add |
| `Predicate<T>` | boolean test(T t) | Collection::isEmpty |
| `Function<T, R>` | R apply(T t) | Arrays::asList |
| `Supplier<T>` | T get() | Instant::now |
| `Consumer<T>` | void accept(T t) | System.out::println |

- Opertaor 인터페이스는 인수에 숫자에 따라 2개로 나누어져있다.
- Predicate 인터페이스는 인수를 하나 받아 boolean 을 반환 한다.
- Function 인터페이스는 인수와 반환타입이 다르다.
- Supplier 인터페이스는 인수를 받지 않고 반환 한다.
- Consumer 인터페이스는 인수를 받고 반환 없이 (인수를) 소비한다.

각 인터페이스는 기본 타입 int, long, double 용으로 각 3개씩 변형이 생겨난다.

Operator : `IntOperator`, `LongOperator` ...

Function 인터페이스는 기본 타입을 반환하는 변형이 9개 더있다. 입력과 결과 타입 결과가 다르기 때문이다. 나머지는 입력이 객체 참조이고 int, long, double 인 변형이 있다.

Function : `LongToIntFunction` (Long 값을 Int 로 반환 받는다. : SrcToResult), `ToLongFunction<int[]>` (int[] 를 Long 으로 반환 받는다.)

기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.

`BiPredicate<T, U>`, `BiFunction<T, U, R>`, `BiConsumer<T, U>` 다.

BiFunction 는 세 변형이 추가로 있다. (`ToIntBiFunction<T, U>`, `ToLongBiFunction<T, U>`, `ToDoubleBiFunction<T, U>`)

Consumer 는 객체와 기본 타입을 받는 변형이 있다. (`ObjDoubleConsumer<T>`, `ObjIntConsumer<T>`, `ObjLongConsumer<T>`)

BooleanSupplier 는 boolean 을 반환하도록한 변형이다.

## 너무 많아

솔직히 외우기엔 수도 많고 어렵다. 그러나 실무에서 자주 쓰이는 함수형 인터페이스 중 상당수를 제공한다.

**표준 함수형 인터페이스는 대부분 기본 타입만 지원하는데 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지 말자.** 동작은 지장이 없지만 계산량이 많은 부분에서 처참한 성능을 보여준다.

## 새로 작성해야 하는 경우

표준 인터페이스에서 필요한 용도가 없다면 직접 작성해야한다. 이 경우 자신이 작성하는 것이 인터페이스임을 명심해야한다. 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 어노테이션을 사용하라.

마지막으로 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메소드들을 다중 정의해서는 안된다. 피하기 쉬운 방법은 다중 정의를 피하는 것이다. (이후에 다룬다.)
