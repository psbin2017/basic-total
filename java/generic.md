# 제네릭

Java 5부터 추가된 타입으로 제네릭은 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거할 수 있다. 클래스와 인터페이스, 그리고 메소드 정의시 타입을 파라미터로 사용한다. 타입 파라미터는 코드 작성 시 구체적인 타입으로 대체되어 다양한 코드 생성에 이점이 있다.

- 강한 타입체크: 잘못된 타입을 제거하기 위해 제네릭 코드에 대해 강한 타입 체크한다.
- 타입 변환제거: 불필요한 타입변환은 성능에 악영향을 미친다. 저장되는 요소를 N타입으로 국한 하여 프로그램 성능 향상을 도모한다.

## 제네릭 타입

> `class<T>`, `interface<T>`

```java
public class ClassName<T> { 
    // ...
}

public interface InterfaceName<T> { 
    // ...
}
```

제네릭 타입은 클래스 또는 인터페이스 이름 뒤에 <> 가 붙고, 사이에 타입 파라미터가 위치한다.

```java
public class Box {

    private Object object;

    public void set(Object object) { this.object = object; }

    public Object get() { return object; }
}
```

Box 클래스는 접근 마다 매 번 타입변환이 발생한다. 빈번한 타입변환은 성능 저하를 야기한다.

```java
public class Box <T>{

    private T t;

    public void set(T) { this.T = T; }

    public T get() { return T; }
}
```

Object 를 모두 제네릭 `T` 로 대체하였다.

```java
public class Box <String> {

    private String t;

    public void set(String t) { this.t = t; }

    public String get() { return t; }
}
```

`Box<String> box = new Box<String>();` 타입 파라미터 `T` 는 `String` 으로 변경되어 재구성된다. 타입 변환이 필요하지 않게 되어 성능 향상을 제공한다.

타입 파라미터로 배열을 생성하려면 `(T[])(new Object[n])` 로 생성한다.

### 자주 사용하는 타입 인자

제네릭 클래스를 만들 때 T 키워드를 사용하였다. T 는 Type 을 의미하는데 타입 인자는 다른 사람이 알아보기 쉽게 만드는 것이 좋다.

| 타입 인자 | 주로 사용되는 의미|
| --- | --- |
| `E` | Element |
| `K` | Key |
| `N` | Number |
| `T` | Type |
| `V` | Value |
| `R` | Result |

### 멀티 타입 파라미터

> `(class<K, V , …>, interface<K, V, …>)`

두 개 이상의 멀티 타입 파라미터를 사용할 수 있다. 각 타입 파라미터를 콤마로 구분한다.

```java
Product<Tv, String> product1 = new Product<Tv, String>();

// 자바 7부터  다이아몬드 연산자를 사용해서 다음과 같이 간단하게 작성할 수 있다.
Product<Car, String> product2 = new Product<>();
```

### 제네릭 메소드

> `(<T, R> R method(T t))`

매개 타입과 리턴타입으로 타입 파라미터를 가지는 메소드이다. 타입 앞에 <> 기호를 추가하고 타입 파라미터를 기술한다.

```java
public <T> Box <T>  boxing(T t) {
    // ... 
}
```

1. 타입 파라미터 T 를 기술
2. 리턴타입으로 제네릭 타입 Box `<T>`
3. 매개변수 타입으로 T 를 기술
4. 지정과 추정

코드에서 타입 파라미터의 구체적인 타입을 명시적으로 지정한다.

> 리턴 타입 변수 = <구체적인 타입>메소드명(매개값);

컴파일러가 매개값의 타입을 보고 구체적인 타입을 추정한다.

> 리턴타입 변수 = 메소드명(매개값);

### 제한된 타입 파라미터

> `<T extends 최상위타입>`

타입 파라미터에는 구체적인 타입을 제한할 필요가 있다.

```java
public <T extends 상위타입> 리턴타입 메소드(매개변수, …) { 
    // ...
}
```

1. `extends` 키워드를 붙여 상위타입을 명시한다.
2. 상위타입은 클래스 뿐만 아니라 인터페이스도 가능하다. (인터페이스도 `extends`)
    1. 구체적인 타입은 상위 타입
    2. 구체적인 타입은 상위 타입의 하위
    3. 구체적인 타입은 구현 클래스
3. 메소드 중괄호 안에서 타입 파라미터 변수로 사용하는 것은 가능한 것은 상위 타입의 멤버로 제한

### 와일드카드 타입

> `<?>`, `<? extends …>`, `<? super …>`

코드에서 일반적으로 ? 를 와일드 카드라고 한다.

- `제네릭타입<?>` : Unbounded Wildcards (제한 없음)
- `제네릭타입<? extends 상위타입>` : Upper Bounded Wildcards (상위 클래스 제한)
- `제네릭타입<? super 하위타입>` : Lower Bounded Wildcards (하위 클래스 제한)

### 제네릭 타입의 상속과 구현

제네릭 타입을 상속해서 자식 제네릭 타입이 될 수 있다. 자식은 추가적으로 타입 파라미터를 가질 수 있다. 인터페이스도 가능하고 인터페이스의 구현 클래스도 제네릭타입이어야 한다.
