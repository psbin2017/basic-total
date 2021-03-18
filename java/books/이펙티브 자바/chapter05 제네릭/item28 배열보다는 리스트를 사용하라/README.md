# ✨ 아이템 28: 배열보다는 리스트를 사용하라

- 배열 : 공변, `Sub` 가 `Super` 의 하위 타입이라면 배열 `Sub[]` 는 `Super[]` 의 하위 타입이 된다.
- 제네릭 : 불공변, 서로 다른 타입 `Type1` 과 `Type2` 이 있다면 `List<Type1>` 과 `List<Type2>` 는 하위도 상위도 아니다.

## 문법상의 허용 차이

```java
// 런타임 실패
Object[] objectArray new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다." // ArrayStoreException

// 컴파일 불가능
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.")
```

어느 쪽이든 Long 저장소에 String 을 넣을 수는 없다. 다만 시점은 다르다는 것은 이전 아이템에서도 여러 번 언급하였다.

이 밖에도 배열의 타입은 실체화 되는 것과 제네릭의 타입은 런타임시 소거된다는 점에서 차이점을 가지고 있으며(소거는 이전 레거시 코드와의 호환성을 위해 이루어진 방법이다.) 이는 이전 제네릭 타입이 매개변수화 타입, 타입 매개변수로 사용할 수 없다는 점에서 다루었다. (`new List<E>[]`, `new List<String>[]`, `new E[]` 처럼 작성시 컴파일 시점에 제네릭 배열 생성 오류를 발생하는 이유이다.)

제네릭 배열을 막은 이유는 타입이 안전하지 않기 때문이다. 이를 허용하면 제네릭의 이유인 런타임에서의 ClassCastException 이 발생하는 것을 막는다는 의미가 없어진다.

```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42); // (2)
Object[] objects = stringLists; // (3)
object[0] = intList; // (4)
String s = stringLists[0].get(0); // (5)
```

위 코드가 된다는 가정하로 ...

1. (2) 는 원소 하나인 `List<Integer>` 를 생성한다.
2. (3) 은 `List<String>` 의 배열을 Object 배열에 할당한다. (공변을 지원, String[] 은 Object[] 의 하위 타입)
3. (4) 는 (2) 에서 생성한 `List<Integer>` 의 인스턴스를 Object 의 첫 원소로 지정한다. 제네릭은 소거 방식으로 구현되어 이 또한 성공한다. (런타임 시점에 `List<Integer>` 가 `List`)
4. 배열에는 지금 `List<Integer>` 인스턴스가 저장되어 있다. 이 때 첫 원소를 꺼내면서 자동으로 String 으로 반환하는데 이 원소는 Integer 이므로 런타임에 ClassCastException 발생

위 모든 오류를 잡는 방법은 첫 상황인 제네릭 배열이 생성되지 않도록 컴파일 오류를 발생시켜야 한다.

소거 메커니즘으로 매개변수화 타입 가운데 실체화될 수 있는 타입은 `List<?>` 와 `Map<?, ?>` 과 같은 비한정적 와일드카드 타입 뿐이다. 배열을 비한정적 와일드카드 타입으로 만들 수 있지만 쓰일일이 거의 없다.

## 배열에서 제네릭으로

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

위 경우 Object 를 원하는 타입으로 형변환 해주어야 하며 혹시나 다른 타입이 들어있으면 형변환 오류가 난다.

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        choiceArray = choices.toArray();
    }
}
```

이 클래스를 컴파일시 오류를 발생시키는데, Object[] 배열을 T[] 로 반환하면 된다. 그 다음에 컴파일시 경고를 발생시키는데 이는 소거 방식으로 인해 런타임에 T 가 무슨 타입인지 보장할 수 없다는 메시지다. 앞서 본 어노테이션으로 해소 가능하지만 배열을 리스트로 바꾸면 더 쉽게 해결할 수 있다.

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

조금 느릴지언정 `ClassCastException` 이 발생할 수 없고 비검사 형변환 경고 또한 반환하지 않는다.
