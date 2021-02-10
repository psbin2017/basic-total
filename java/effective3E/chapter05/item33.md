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

## ✨ 아이템 33: 타입 안전 이종 컨테이너를 고려하라

제네릭은 자주 사용하는 컬렉션과(`Set<E>`, `Map<K, V>`) 단일원소 컨테이너로(`ThreadLocal<T>`, `AtomicReference<T>`) 쓰인다. 이런 쓰임에 매개변수화되는 대상은 원소가 아닌 컨테이너 자신이다. 이는 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한됨을 의미한다.

그러나 유연한 수단이 필요한 경우가 있다. 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 이 설계를 타입 안전 이종 컨테이너 패턴이다.

```java
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type);
}
```

```java
Favorites f = new Favorites();

f.putFavorite(String.class, "Java");
f.putFavorite(Integer.class, 100);
f.putFavorite(Class.class, Favorites.class);

String fString = f.getFavorite(String.class);
Integer fInteger = f.getFavorite(Integer.class);
Class<?> fClass = f.getFavorite(Class.class);
```

`Favorites` 인스턴스 타입 안전하다. String 을 요청하여 Integer 를 반환하는 일은 절대 없다. 또한 맵과 달리 여러 가지 타입 원소를 담을 수 있다.

## 이종 컨테이너 패턴

```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

키가 와일드 카드 인점에 주목하자. `Class<String>`, `Class<Integer>` 식으로 되어 다양한 타입을 지원하는 힘을 가진다.

다음 값이 Object 인 것은 해당 맵은 키와 값 사이의 타입 관계를 보증하지 않는다는 말이다. 모든 값이 키로 명시한 타입임을 보증하지 않는다.

`putFavorite` 는 주어진 객체와 인스턴스를 `favorites` 에 추가하는 관계를 만들고 끝난다. 키와 값 사이의 타입 링크 정보는 버려진다. 즉, 그 값이 그 키 타입의 인스턴스라는 정보가 사라진다. 하지만 `getFavorite` 에서 이 관계를 다시 찾을 수 있으니 상관 없다.

`getFavorite` 는 Class 의 cast 메서드를 사용해 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다. cast 메서드는 형변환 연산자의 동적 버전이다. 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사하고 맞으면 그대로 인수를 반환하고 아니면 ClassCastException 을 던진다.

첫 번째 제약으로, 악의적으로 Class 객체를 다른 타입으로 넘기면 타입 안전성이 깨진다. 그러나 이 경우 클라이언트 코드에서 컴파일 시점에 비검사 경고가 뜰 것이다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.case(instance));
}
```

컬렉션에서 `checkedSet`, `checkedList`, `checkedMap` 과 같은 메서드가 있는데 이 방식을 사용한 컬랙션 래퍼이다.

두 번째 제약으로, 실체화 불가 타입에는 사용할 수 없다. `String`, `String[]` 은 가능하지만 `List<String>` 은 저장할 수 없다. 이 경우 `List<String>` 용 Class 를 얻을 수 없기 때문이다. `List<String>` 과 `List<Integer>` 는 같은 `List.class` Class 객체를 공유하기 때문에 불가능하다.

### 슈퍼 타입 토큰(super type token)

```java
Favorites f = new Favorites();
List<String> pets = Arrays.asList("개", "고양이", "앵무");

f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
```

두 번째 제약을 우회하는 방법인 슈퍼 타입 토큰을 사용하는 방법이다. 슈퍼 타입 토큰을 사용하면 제네릭 타입을 넣을 수 있게 된다. (많은 라이브러리에서 이를 제공한다.)

## 한정적 타입 토큰

`Favorites` 는 어떤 Class 객체든 받아들인다. 때때로 이를 허용하는 타입을 제한하고 싶을 때가 있는데, 한정적 타입 매개변수나 한정적 와일드카드를 사용하면 표현 가능한 타입을 제한하는 타입 토큰이다. 어노테이션 API 는 한정적 타입 토큰을 적극적으로 사용한다.

```java
public <T extends Annontation> T getAnnontation(Class<T> annontationType);
```

명시한 타입의 어노테이션이 달려있다면 그 어노테이션을 반환하고, 없다면 null 을 반환한다. 즉, 어노테이션된 요소는 그 키가 어노테이션 타입인 타입 안전 이종 컨테이너인 것이다.
