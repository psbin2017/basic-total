# ✨ 아이템 54: null이 아닌 빈 컬렉션이나 배열을 반환하라

때로는 빈 컨테이너를 할당하는 데도 비용이 드니 null 을 반환하는게 좋다는 주장이 있다.

1. 이 할당이 성능 저하의 주범이라고 확신되지 않는 한 이 정도의 성능차이는 신경 쓸 수준이 못된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

사용 패턴에 따라서 빈 컬렉션 할당이 문제가 될 수 있다. 빈 "불변" 컬렉션을 반환하는 것으로 이를 해소할 수 있다.

```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```

배열도 마찬가지다. null 이 아닌 길이가 0 인 배열을 반환하지 않고 "불변" 배열을 만들어놓고 반환하는 방법이 있다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
