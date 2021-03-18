# ✨ 아이템 43: 람다보다는 메서드 참조를 사용하라

> 간결함 기준 : 메소드 참조(Method Reference) > 람다 > 익명 클래스

```java
// 람다
map.merge(key, 1, (count, incr) -> count + incr);

// 메소드 참조
map.merge(key, Integer::sum)
```

단, 이런 람다는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기 떄문에 필요애 따라 람다로 적용하는 방법이 바람직하다. IDE 들은 람다를 메소드 참조로 대체할 것을 권할 것이다. 보통은 득이 되지만 항상 득이 되는 것은 아니다.

```java
// 메소드 참조
service.execute(GoshThisClassNameIsHumongous::action);

// 람다
service.execute(() -> action());
```

## 메소드 참조 유형

| 유형 | 예 | 람다 |
| --- | --- | --- |
| 정적 | `Integer::parseInt` | `str -> Integer.parseInt(str)` |
| 한정적 (인스턴스) | `Instance.now()::isAfter` | `Instance then = Instance.new(); t -> then.isAfter()` |
| 비한정적 (인스턴스) | `String::toLowerCase` | `str -> str.toLowerCase()` |
| 클래스 생성자 | `TreeMap<K, V>::new` | `() -> new TreeMap<K, V>()` |
| 배열 생성자 | `int[]::new` | `len -> new int[len]` |

우선 인스턴스 참조 유형은 크게 2가지다.

1. 수신 객체(receiving object; 참조 대상의 인스턴스)를 특정하는 한정적 인스턴스 메소드 참조
2. 수신 객체를 특정하지 않는 비한정적 인스턴스 메소드 참조

한정적 참조는 정적 참조와 비슷하다. 함수 객체가 받는 인수와 참조되는 메소드가 받는 인수가 동일하다.

비한정적 참조는 함수 객체를 사용하는 시점에 수신 객체를 알려준다. 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에서 사용된다.

마지막으로 클래스 생성자를 가리키는 참조와 배열 생성자를 가리키는 참조가 있다.

### 정적 메소드 참조

> `x -> Integer.parseInt(x)`

파라미터 x 를 인자로 사용한다. `x.length()`, `x.toUpperCase()` ... 등과 같은 사용을 하지 않았다.

### 한정적 메소드 참조

```java
// 메소드 참조
Instance.now()::isAfter

// 메소드 참조2
Instance then = Instance.now();
then::isAfter;

// 람다
Instance then = Instance.now();
t -> then.isAfter();
```

> 외부에서 선언된 객체의 메소드를 호출하거나 객체를 생성한다. (한정적: 객체가 특정되어 제한된다는 의미)

### 비한정적 메소드 참조

```java
// 메소드 참조
String::toLowerCase;

// 람다
(String str) -> str.toLowerCase();
```

1. 객체의 생성을 파라미터로 제공한다.
2. 람다 표현식 내부에서 객체 생성이 생기며 String 이라는 수신 객체를 알게 된다.
3. 메소드 참조로 해석하면 `str::toLowerCase` 라고 표현할 수 없기 때문에 String 이라는 수신 객체를 알려 나타낸다.
