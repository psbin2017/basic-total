# ✨ 아이템 55: 옵셔널 반환은 신중히하라

자바 8 이전에는 특정 조건에 값을 반환할 수 없을때 선택지는 2가지였다.

1. 예외를 던진다.
2. null 을 반환한다.

그러나 둘 다 허점이 있다.

1. 예외는 진짜 예외적인 상황에만 사용하는게 적절하다.
2. null 로 반환하면 어딘가에서는 별도의 null 처리 코드를 추가해야한다.

자바 8 로 올라가면서 하나의 선택지가 추가되었다. `Optional<T>` 는 null 아닌 T 타입 참조를 하나 담거나 혹은 아무 것도 담지 않을 수 있다.

1. 참조가 있으면 비어있지 않다.
2. 참조가 없으면 비어있다.

## Optional

옵셔널은 원소를 최대 1개만 가질 수 있는 "불변" 컬렉션이다. 옵셔널을 반환하는 메소드는 예외를 던지는 메소드보다 유연하고 사용하기 쉬우며, null 을 반환하는 메소드보다 오류 가능성이 작다.

[OptionalReturn](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/method/OptionalReturn.java)

**옵셔널을 반환하는 메소드는 절대 null 을 반환하지 말자.** 옵셔널 반환을 선택하는 기준은 무엇일까?

**옵셔널은 검사 예외와 취지가 비슷하다.** 있을 수도 없을 수도 있음을 API 사용자게에 명확하게 알려준다.

### 활용

```java
.orElse("단어가 없다.");
.orElseThrow(IllegalArgumentException::new);
```

값이 비어있을 때 기본 값을 정해주거나, 지정한 예외를 발생시킬 수도 있다. 기본값을 설정하는 비용이 커서 부담이 될 때가 있는데 `orElseGet()` 을 사용하면 값이 처음 필요할 때 `Supplier<T>` 를 사용하여 생성하므로 초기 설정 비용을 낮출 수
있다.

적합한 메소드를 찾지 못했다면 `isPresent()` 로 대체하여 사용할 수 있다.

스트림과 사용하면 옵셔널들을 `Stream<Optional<T>>` 로 받아, 그 중 채워진 옵셔널들에서 값을 뽑아 `Stream<T>` 에 건네 담아 처리하는 경우가 많다.

```java
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get);
```

자바 9 에서는 `Optional` 에 `stream()` 메소드가 추가되었다. `Optional` 을 `Stream` 으로 변환해주는 어답터이다.

```java
streamOfOptionals
    .flatMap(Optional::stream)
```

## 부작용

컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.

`Optional<List<T>>` 와 같이 사용하면 안된다는 뜻이다. `List<T>` 로 반환하는게 좋다.

빈 컨테이너를 그대로 반환하면 클라이언트에서 옵셔널 처리 코드를 넣지 않아도 된다.

> 마치 null 피하려고 Optional 을 피하는거랑 맥락이 비슷하다.

## 언제 사용해야 할까

어떤 경우에 Optional 을 반환해야할까?

**결과를 얻을 수 없으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 `Optional<T>` 로 반환한다.**

`Optional<T>` 도 새로 객체를 할당하고 메소드를 호출하기에 비용이 든다. 그래서 성능이 중요한 곳에서는 옵셔널이 맞지 않을 수도 있다.

박싱 기본 타입을 담는 옵셔널은 값을 두 겹이나 감싸기 때문에 `OptionalInt`, `OptionalLong`, `OptionalDouble` 를 사용하면 된다. 따라서 **박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.**

1. 옵셔널을 컬렉션의 키, 값
2. 배열의 원소

로 사용하는게 적절한 상황은 거의 없다.

### 필드

옵셔널을 인스턴스 필드에 저장해두는 게 필요한 경우가 있을까? 이 경우 선택적 필드의 게터 메소드들이 옵셔널을 반환하게 해주면 좋은 방법일 수도 있다. 이 때 필드 자체를 옵셔널로 선언하는 것도 좋은 방법이될 것이다.
