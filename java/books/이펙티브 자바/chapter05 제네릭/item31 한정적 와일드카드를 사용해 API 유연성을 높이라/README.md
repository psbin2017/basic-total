# ✨ 아이템 31: 한정적 와일드카드를 사용해 API 유연성을 높이라

이전에 말한 불공변을 곰곰히 따져보자. `List<Type1>` 과 `List<Type2>` 은 하위 타입도 상위 타입도 아니다. `List<Object>` 에는 어떤 객체든 넣을 수 있지만, `List<String>` 은 `List<Object>` 가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.

하지만 때론 불공변 방식보다 유연한 방식이 필요하다. 한정적 와일드카드 타입은 특별한 매개변수화 타입을 지원한다.

## `pushAll`

```java
// before
// 논리적으로 Number 가 Integer 하위이니 Iterable<Integer> 를 제공시 가능할 것처럼 보이지만 불공변으로 오류메시지가 뜬다.
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}

// after
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(e);
    }
}
```

메시지는 분명하다 유연서을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.

## `popAll`

```java
// before Set<Number> 의 원소를 Object 컬렉션으로 옮길 때 불공변 오류가 발생
public void popAll(Iterable<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}

// after
// 입력 매개변수 타입이 E 의 Collection 이 아니라 E 의 상위 타입의 Collection 이어야한다.
public void popAll(Iterable<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```

## PECS

> 펙스(PECS): producer-extends, consumer-super

다음 공식을 외워 두면 어떤 와일드카드 타입을 써야하는지 기억하는 데 도움이 된다.

**매개변수화 타입이 T 생산자라면 `<? extends T>`**

**매개변수화 타입이 T 소비자라면 `<? super T>`**

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    // ...
}

public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
    // ...
}
```

반환 타입은 여전히 `Set<E>` 임에 주목한다. 반환타입에 한정적 와일드 카드를 사용하면 클라이언트 코드에서도 와일드카드 타입을 써야하기 때문에 유연성을 높이기는 커녕 복잡도가 증가한다.

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
// in java 8
Set<Number> numbers = union(integers, doubles);

// in java 7
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

제대로 사용하면 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못한다. 클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API 에는 문제가 있을 가능성이 크다.

일반적으로 `Comparable` 은 언제나 소비자이므로 `Comparable<? super E>` 를 사용하는 편이 낫다. 반대로 `Comparator` 는 `Comparator<? super E>` 를 사용하는 편이 낫다.

## 선언

**메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.** 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드 카드로 바꾼다.

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

private static <E> void swapHelper(List<E> list, int i, int j) {
    // List<?> 에는 null 외에는 어떤 값도 넣을 수 없다.
    // 따라서 와일드카드 타입의 실제 타입을 알려주는 메소드로 형변환 없이 실제 타입으로 알아낼수 있다.

    // swapHelper 메소드는 리스트가 List<E> 임을 알고 있다.
    // 항상 E 임을 보장하므로 리스트에 넣어도 안전함을 보장한다.
    list.set(i, list.set(j, list.get(i)));
}
```
