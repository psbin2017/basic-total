# Optional

> 규칙 #1 : Optional 변수 또는 return 값에 null 을 사용하면 안된다.
> 규칙 #2: Optional 이 있다는 것을 증명할 수 없다면 `Optional.get()` 사용하지 않는다.
> 규칙 #3: `Optional.isPresent()` 와 `Optional.get()` 에 대한을 선호하라.
> 규칙 #4: 일반적으로 값을 얻기위해 메소드를 연결하는 특정 목적을 위해서 Optional 을 만드는 것은 부적절하다.
> 규칙 #5: Optional 체이닝이 너무 중첩되거나 `Optional<Optional<T>>` 의 중간 결과가 있는 경우 복잡해진다.
> 규칙 #6: 필드, 메소드 매개 변수 및 컬렉션에서 선택사항을 사용하지마라.

[Optional - The Mother of All Bikesheds by Stuart Marks](https://www.youtube.com/watch?v=Ej0sss6cq14)

`Optional<T>` Java8 에 도입되었다.

`Optional<T>` 은 두 가지 상태 중 하나를 가진다.

- T 에 대한 null 이 아닌 참조를 포함합니다. 이를 `present` 라고 부른다.
- 비었다, 이를 `absent` 라고 부르며 null 과는 다른 상태이다.

Primitive 타입 특수화

- Optional 자체는 참조 타입이다. 그리고 Optional 이 null 일 수 도 있지만 **DON'T**

## Optional 의 유용함

특정 ID 를 가진 고객에 대한 `List<Customer>` 검색을 구현한다고 하자. 최초의 API 는 `Stream.search(Predicate)` 이었다.

```java
Customer customerByID(List<Customer> custList, int custID) {
    return custList.stream()
                    .search(c -> c.getID() == custID);
}
```

조건(predicate)과 일치하는 요소가 스트림에 없으면 어떻게 되는가? 아마도 `search()` 는 null 을 반환 할 것이다. 그리고 `customerByID()` 도 null 을 반환할 것이다.

### return the Customer name

```java
String customerNameById(List<Customer> customerList, int custId) {
    return customerList.stream()
                        .filter(c -> c.getCustId() == custId)
                        .findFirst()
                        .get()
                        .getName(); // Customer 를 찾을 수 없는 경우 NullPointerException
}
```

다음으로 변경해야 한다.

```java
String customerNameByIdStep2(List<Customer> customerList, int custId) {
    Optional<Customer> optionalCustomer = customerList.stream()
                        .filter(c -> c.getCustId() == custId)
                        .findFirst();

    return optionalCustomer.isPresent() ? optionalCustomer.get().getName() : "UNKNOWN";
}
```

## Optional 의 해석

Optional 은 "결과 없음" 을 분명하게 나타낼 필요가 있고 null 을 사용하면 **오류가 발생할 가능성이 큰** 경우 라이브러리 메소드 **반환 유형**에 대한 **제한된** 메커니즘을 제공하기 위한 것이다.

### Optional 을 사용한 재방문(Revisiting) 예시

실제 스트림 API 에는 `findFirst()` 및 `findAny()` 가 있다. 조건(predicate) 은 `filter()` 업스트림을 통해 전달되어야 한다.

```java
String customerNameByID(List<Customer> custList, int custID) {
    Optional<Customer> opt = custList.stream()
                                        .filter(c -> c.getID() == custID)
                                        .findFirst();
    // To get the value from an Optioanl, call get()
    return opt.get().getName();
}
```

그러나 Optional 이 비어 있으면 `get()` 은 `NoSuchElement-Exception` 을 발생시키며 이는 거의 개선되지 않았다!

## Optional 에서 안전하게 값 얻기

### get()

```java
String customerNameByIdStep1(List<Customer> customerList, int custId) {
    Optional<Customer> optionalCustomer = customerList.stream()
                        .filter(c -> c.getCustId() == custId)
                        .findFirst();
    // 이것은 안전하지만 null 을 확인하는 것보다 좋지 않다.
    return optionalCustomer.get().getName();
}
```

```java
String customerNameByIdStep2(List<Customer> customerList, int custId) {
    Optional<Customer> optionalCustomer = customerList.stream()
                        .filter(c -> c.getCustId() == custId)
                        .findFirst();

    return optionalCustomer.isPresent() ? optionalCustomer.get().getName() : "UNKNOWN";
}
```

> 규칙 #2: Optional 이 있다는 것을 증명할 수 없다면 `Optional.get()` 사용하지 않는다.

불행하게도 이는 사람들이 `get()` 전에 `isPresent()` 를 테스트하도록 유도한다.

> 규칙 #3: `Optional.isPresent()` 와 `Optional.get()` 에 대한을 선호하라.

## Example: orElse() Family

```java
// orElse(default)
// 값이 있는 경우 반환하고 아니면 기본 값을 반환한다.
Optional<Data> opt = ...
Data data = opt.orElse(DEFAULT_DATA);

// orElseGet(supplier)
// 값이 있는 경우 반환하거나 공급자(supplier) 를 호출하는 기본 값을 가져와 반환한다.
Optional<Data> opt = ...
Data data = opt.orElseGet(Data::new);

// orElseThrow(exsupplier)
// 값이 있는 경우 반환하거나 공급자(supplier) 로 부터 얻은 예외를 던진다.
Optional<Data> opt = ...
Data data = opt.orElseThrow(IllegalStateException::new)
```

## Example: map()

```java
String customerNameByIdStep3(List<Customer> customerList, int custId) {
        Optional<Customer> optionalCustomer = customerList.stream()
                            .filter(c -> c.getCustId() == custId)
                            .findFirst();

        /**
         * map() - present, 값을 다른 값으로 변환하거나 매핑하고 결과를 Optional 로 반환한다.
         * 그렇지 않으면 빈 Optional 을 반환한다.
         *
         * orElse() 는 map() 호출의 결과에서 직접 연결되어 값이 있는 경우 추출할 수 있다.
         */
        return optionalCustomer.map(Customer::getName).orElse("UNKNOWN");
    }
```

```java
String customerNameByIdStep4(List<Customer> customerList, int custId) {
    return customerList.stream()
                        .filter(c -> c.getCustId() == custId)
                        .findFirst()
                        /**
                         * Optional 에 대한 map() 그리고 orElse() 호출은 스트림 파이프 라인의 끝에서 직접 연결될 수 있다.
                         */
                        .map(Customer::getName)
                        .orElse("UNKNOWN");
}
```

## Example: filter()

(OpenJDK Layer.java 의 일부 특허로 조정)

설정 객체(Configuration)가 주어지면 부모 설정이 있는지 확인한다. 이는 레이어의 구성과 동일하다.

```java
config.parent()
        /**
         * filter() - 없으면 비어 있다.
         * present, 값에 서술(predicate)을 적용하여, 참이면 값을 반환하고 거짓이면 비어있다.
         */
        .filter(config -> config == this.config())
        .orElseThrow(IllegalArgumentException::new);
```

## Example: ifPresent()

```java
/**
* ifPresent() - present, 값에 대한 람다(소비자(Consumer)) 를 살행하고 그렇지 않으면 아무 작업도 하지 않는다.
*/
getTask(...).ifPresent(task -> executor.runTask(task));

// best case: 메소드 레퍼런스
getTask(...).ifPresent(executor::runTask);
```

## 추가 메소드

- 정적 팩토리 메소드
  - `Optional.empty()` - 비어있는 Optional 반환
  - `Optional.of(T)` - T 를 포함하는 현재 Optional 을 반환
    - T 는 null 이 아니다.
- `flatMap(Function<T, Optional<U>>)`
  - `map()` 과 비슷하지만, Optional 을 반환하는 함수(Function)를 사용하여 변환한다.
- `Optional.equals()` and `hashCode()` - 예상대로 대부분
- 테크닉: Optional 을 반환하는 메소드 단위 테스트

```java
assertEquals(Optional.of("expected value"), optionalReturningMethod());
assertEquals(Optional.empty(), optionalReturningMethod());
```

## Example: Stream of Optional

`List<Customer>` 에서 해당 되는 `List<Customer>` 로 변환.

`findByID()` 이 `Optional<Customer>` 를 반환한다고 가정한다.

```java
// Java 8
List<Customer> list = custIDlist.stream()
                                .map(Customer::findByID)
                                .filter(Optional::isPresent) // Optional 만 제시한다.
                                .map(Optional::get) // 여기에서 값을 추출한다.
                                .collection(Collectors.toList());

// Java 9
List<Customer> list = custIDlist.stream()
                                .map(Customer::findByID)
                                .flatMap(Optional::stream) // Optional.stream() 을 사용하면 filter() & map() 를 flatMap() 에 통합가능하다.
                                .collect(Collectors.toList());
```

## Example: Null 과 Optional 간 조정

- 때로는 null 을 원하는 코드에 Optional 사용 코드를 적용해야 하거나 이 반대의 경우도 마찬가지이다.
- nullable 참조가 있고 선택사항이 필요한 경우

`Optional<T> opt = Optional.ofNullable(ref)`

Optional 이 있고 nullable 참조가 필요한 경우

`opt.orElse(null)` - 이 경우가 아니라면 일반적으로 `orElse(null)` 를 피해야 한다.

## 사용, 남용, 요용

### 메소드 체이닝 남용

> 규칙 #4: 일반적으로 값을 얻기위해 메소드를 연결하는 특정 목적을 위해서 Optional 을 만드는 것은 부적절하다.

```java
// 좋지 않음
String process(String s) {
    return Optional.ofNullable(s).orElseGet(this::getDefault);
}

// null 을 확인하는 게 좋음
String process(String s) {
    return (s != null) > s : getDefault();
}
```

### Avoiding If-Statements is Cool, But

```java
Optional<BigDecimal> first = getFirstValue();
Optional<BigDecimal> second = getSecondValue();

// 첫 번째와 두 번째를 추가하여 비어있는 값을 0 으로 처리하고, 둘 다 비어 있지 않으면 합계의 Optional 을 반환한다. 이 경우 빈 Optional을 반환한다.

// 영리하고 원하는 수의 옵션을 결합 할 수 있다.
Optional<BigDecimal> result = Stream.of(first, second)
                                    .filter(Optional::isPresent)
                                    .map(Optional::get)
                                    .reduce(BigDecimal::add);

// 더 영리한 방법, 이 것이 올바른지 확인해야한다.
Optional<BigDecimal> result2 = first.map(b -> second.map(b::add))
                                    .orElse(b)
                                    .map(Optional::of)
                                    .orElse(second);

// 규칙 #5: Optional 체이닝이 너무 중첩되거나 Optional<Optional<T>> 의 중간 결과가 있는 경우 복잡해진다.
Optional<BigDecimal> result3;
if ( ! first.isPresent() && ! second.isPresent() ) {
    result = Optional.empty();
} else {
    result  = Optional.of(first.orElse(BigDecimal.ZERO).add(second.orElse(BigDecimal.ZERO)) );
}

```

### Optional.get() 의 문제

`get()` "매력적이면서 성가신" 녀석이다.

- 짧은 이름이 나타내는 것보다 훨씬 덜 유용하다.
- 그것을 지키는 것을 잊기 쉽다.
- 불량한 `isPresent()` / `get()` 코딩 스타일로 오해되기 쉽다.
- `get()` 은 많은 경우로 오용됩니다 => 따라서 이는 잘못된 API 가 된다.

문제 해소 계획

- `get()` 대체물 도입
- `get()` 사용 중단, (제거 용이 아님)
- 도입된 경고로 인해 보류된 지원 중단

> 규칙 #2: Optional 이 있다는 것을 증명할 수 없다면 `Optional.get()` 사용하지 않는다.

그리고

> 규칙 #3: `Optional.isPresent()` 와 `Optional.get()` 에 대한을 선호하라.

### Optional 을 사용하지 않는 영역

필드에 Optional 을 사용하지 마라.

- 초기화시 대체 값을 입력하라. "null Object" 패턴을 사용해라. 아니면 null 을 사용해라

Optional 에서 메소드 매개 변수를 사용하지 마라

- 매개 변수(parameters)를 선택적으로 만드는 데는 실제로 작동하지 않는다.
- 호출 사이트가 모든 것에 대한 옵션을 생성하도록 한다.

```java
myMethod(Optional.of("some value"));
myMethod(Optional.empty());
```

컬렉션(Collection)에서 Optional 을 사용하지 마라.

- 일반적으로 일종의 디자인 냄새를 나타낸다.
- 종종 더 나은 표현 방법이 없다.

> 규칙 #6: 필드, 메소드 매개 변수 및 컬렉션에서 선택사항을 사용하지마라.

기억하라 Optional 은 상자다.

- 16 바이트나 소모한다.
- 별도의 객체로 잠재적으로 GC 에 영향을 준다.
- 항상 의존성이 필요하여 캐시 누락이 발생한다.
- 하나의 Optional 은 좋지만, 많은 Optional 인스턴스로 데이터 구조를 버리게 되어 성능 문제로 야기된다.

모든 null 을 Optional 로 바꾸지 마라

- null 을 제어하면 오히려 null 이 안전하다.
- 비공개 필드의 null 을 쉽게 확인할 수 있다.
- nullable 매개 변수는 괜찮다. (declasse 하다면)
  - 라이브러리 코드는 인수 확인을 담당해야한다.

*********************************************TODO*********************************************

## Bikeshedding

### Why Isn't Optional Serializable?

Background: Value types - Project Valhalla

- an "object" that has no notion of identity
- "codes like a class, works like an int"
- we eventually want to convert Optional into a value type

Disclaimer from Optional's javadoc:

This is a value-based class; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization) on instances of Optional may have unpredictable results and should be avoided.

> Rule #7: Avoid using identity-sensitive operations on Optionals.

### Serialization Impact on Future Evolution

- JDK rule: **forward and backward** serialization compatibility across releases
- If Optional were serializable today, it would be serialized as an Object
  - It'll always be serialized as an Obejct, even if it eventually becomes a value type
- Serialization inherently depends on object identity
- Consequences of Optional being serializable
  - it might prevent it from being converted into a value type in the future
  - deserializing an Optional might result in a "boxed" value type

### Why Not Use Optional in Fields

More a style issue than a correctness issue

- usually there's a better way to model absence of a value
- use of Optional in fields often arises from slavish desire to eliminate nullable fields
- remember, eliminating nulls isn't a goal of Optional

Using Optional in fields...

- creates another object for every field
- introduces a dependent load from memory on every field read
- clutters up your code
- to what benefit? ability to chain methods?

Colebourne: Optional, A Pramatic Approach

- use nullable fields, getters should return Optional

Ernst: Nothing is Better Than the Optional Type

- use Nullness Checker
- glass is one-quarter full
