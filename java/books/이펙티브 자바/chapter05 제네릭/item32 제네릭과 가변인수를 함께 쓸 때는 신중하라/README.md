# ✨ 아이템 32: 제네릭과 가변인수를 함께 쓸 때는 신중하라

varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.

```text
waarning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
```

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringList;
    objects[0] = intList; // 힙 오염 발생
    String s = stringList[0].get(0) // ClassCastException
}
```

타입 안정성이 깨지며 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다. 제네릭 배열을 비허용하였는데 제네릭 varargs 매개변수 메소드는 왜 선언할 수 있게 해둔걸까? 답은 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메소드가 실무에서 매우 유용하기 때문이다.

자바 7 에서 `@SafeVarargs` 가 추가되어 제네릭 가변인수 메소드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. `@SafeVarargs` 는 메소드 작성자가 그 메소드가 타입 안점함을 보장하는 장치이다.

메소드가 이 배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입이 안전하다. varargs 매개변수 배열이 호출자로부터 그 메소드로 순수하게 인수들을 전달하는 일만 수행한다면 메서드는 안전하다.

```java
// 제네릭 매개변수 배열의 참조를 노출하여 안전하지 못하다.
static <T> T[] toArray(T... args) {
    return args;
}
```

## `@SafeVarargs`

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

제네릭이나 매개변수화 타입 varargs 매개변수를 받는 모든 메소드에 `@SafeVarargs` 를 선언한다. (이 말은 안전하지 않은 varargs 메소드는 절대 작성해서는 안 된다는 뜻이다.) 또한 힙 오염 경고가 뜨는 메소드가 있다면, 그 메소드가 진짜 안전한지 점검하라.

- varargs 매개변수 배열에 아무것도 저장하지 않는다.
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

`@SafeVarargs` 는 재정의 불가능한 메소드에만 달아야한다. 재정의해서 안전할지 보장할 수 없기 때문이다. 때문에 자바 8 에서는 정적 메소드와 final 인스턴스 메소드에만 붙일 수 있고, 자바 9 에서는 private 인스턴스 메소드에도 허용한다.

varargs 매개변수가 아닌 List 매개변수로 바꿀 수도 있다.

```java
static <T> List<T> flatten(List<List<? extends T>>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

선언 부분만 수정하였다. 이 장점은 컴파일러가 메소드 타입 안전성을 검증할수 있으며, 이로 인해 어노테이션을 선언하지 않아도 된다. 클라이언트 코드가 지저분해지는 것과 조금 느려지는 것 이외에는 단점이 없다.
