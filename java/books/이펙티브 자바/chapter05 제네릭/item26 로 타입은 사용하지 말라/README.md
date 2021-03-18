# ✨ 아이템 26: 로 타입은 사용하지 말라

제네릭 타입 : 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 형태 (제네릭 클래스, 제네릭 인터페이스)

로 타입 : 제네릭 타입에서 타입 매개 변수를 전혀 사용하지 않은 형태

`List<E>` 의 로 타입은 `List` 이다. 이는 제네릭이 도래하기전에 호환되도록 하기 위한 방법이다. 로 타입을 사용하게 되면 서로 다른 타입을 사용해도 오류 없이 컴파일되고 실행된다. (컴파일러가 모호한 경고메세지를 알린다.) 이 경우 직접 사용하기 전까지 오류를 알아 챌 수 없다.

오류를 알아채는 가장 이상적인 순간은 컴파일 시점이다. 위 상황은 오류가 발생하고 런타임에야 알아챌 수 있다는 문제이다. **로 타입을 사용하면 제네릭이 가져다주는 안정성과 표현력을 모두 버리는 셈이다.** 단지 이전 코드를 모두 수용해야한다는 호환성으로 로 타입을 지원하게 된 것이다.

## `List<Object>`

`List<Object>` 의 경우 괜찮다. 이 경우 모든 타입을 허용한다는 의미를 명시적으로 컴파일러에게 전달하였기 때문이다.

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // 로 타입: ClassCastException
}

// 로 타입 경고 발생, 컴파일 됨
private static void unsafeAdd(List list, Object o) {
    list.add(o);
}

// List<String> cannot bt converted to List<Object>
private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
}
```

## 와일드카드 타입

원소의 타입을 몰라도 되는 경우 로 타입을 사용하지 않고 와일드카드 타입을 대신 사용하는 것이 좋다. `Set<?>` 가 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 `Set` 타입이다. 로 타입과의 차이는 물음표는 안전하고 로 타입은 안전하지 않다. 로 타입 컬렉션에는 아무 원소나 넣을 수 있어 타입 불변식을 훼손하기 쉽다. 그러나 `Collection<?>` 에는 null 외에 어떤 원소도 넣을 수 없는데 이는 컬렉션 타입 불변식을 훼손하였기 때문에 막은 것이다.

이 제약을 해소하고자 한다면 제네릭 메소드 타입이나 한정적 와일드카드 타입을 사용해야 한다.

## class 리터럴, instanceof

자바 명세에서는 class 리터럴에 매개변수화 타입을 사용하지 못하게 하였다. (`List.class` 는 가능하지만 `List<String>.class` 는 불가능하다는 말이다.)

두 번째 예외는 instanceof 연산자와 관련이 있다. 런타임 시점에는 제네릭 정보가 제거되어 instanceof 연산자에는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용이 불가능하다. 이 경우 로 타입과 비한정적 와일드카드 타입이 똑같이 동작한다.

```java
if (o instanceof Set) { // 로 타입
    Set<?> s = (Set<?>) o; // 와일드카드 타입
    // ...
}
```
