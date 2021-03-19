# ✨ 아이템 46: 스트림에서는 부작용 없는 함수를 사용하라

스트림의 계산은 단계를 가진다. 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수이다. 단계에서 다른 가변 상태를 참조하지 않고 스스로도 다른 상태를 변경하지 않는다. 결과적으로 중간/종단 연산에서 모두 부작용이 없이 제공되어야 한다.

자바 프로그래머라면 for-each 반복문을 빈도수 있게 사용하여 사용할 줄 아는데 `forEach` 연산이 종단 연산과 비슷하게 생겼다. 그러나 종단 연산 중 가장 덜 스트림 답다. 대놓고 반복적이라서 병렬화 할수 없다.

**`forEach` 는 스트림 결과 계산을 보고할 때만 사용하고 계산하는 데는 사용하지 말자.** 대신 수집기(`Collectors`)를 사용하면 스트림 원소를 손쉽게 컬렉션으로 모을 수 있다.

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed)
    .limit(10)
    .collect(toList());
```

## Collectors

### toMap

스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다. **`toMap` 은 스트림의 각 원소가 고유한 키에 매핑되어있을 때 적합하다.**

`groupingBy` 와 `merge` 등으로 원소를 고유 키로 가공하는 방법으로 `toMap` 으로 사용하는 전략도 있다.

인수를 3개 가지는 `toMap` 은 "어떤 키"와 "어떤 키에 대한 원소들 중 하나"를 골라 연관 짓는 맵을 만들때 유용하다.

```java
Map<Artist, Album> topHits = albums.collect(
    toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

> 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다. 이 외에도 last-write-wins 수집기를 만들 때도 유용하다.

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

마지막은 4개의 인수로 마지막 인수를 맵 팩토리를 받는다. 이 인수로는 `EnumMap` 이나 `TreeMap` 처럼 원하는 특정 맵 구현체를 직접 지정 가능하다.

### groupingBy

입력으로 분류 함수를 받고 원소들을 카테고리 별로 모아 놓은 맵을 담은 수집기를 반환한다. 가장 형태가 간단한 것은 분류 함수 하나를 받고 맵을 반환 하는 형태이다.

```java
words.collect(groupingBy(word -> alphabetize(word)));
```

> 알파벳화 한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성한다.

groupingBy 가 반환하는 수집기가 리스트 외의 값을 가지는 맵을 여러개 생성하려 한다면 분류 함수와 역할에 맞는 컬렉션 인수를 주는 방법이 있다. (`toSet()`, `toCollection(collectionFactory)`) 다운 스트림 수집기로 `counting()` 을 건네는 방법도 있다.

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```

세 번째는 맵 팩터리를 지정하는 것이다. 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다. 명시적으로 `TreeSet`, `TreeMap` 을 반환하는 수집기를 만들 수있다는 것이다.

각각에 대응하는 `groupingByConcurrent` 로 메서드의 동시 수행 버전으로 `ConcurrentHashMap` 인스턴스를 반환한다.

`groupingBy` 와 사촌격인 `partitioningBy` 도 있는데 분류 함수자리에 predicate 를 받아 키가 Boolean 인 맵을 반환한다.

## 그 외

`summing`, `averaging`, `summarizing` 으로 시작하며, 각각 int, long 그리고 double 스트림용으로 하나씩 존재한다. 그리고 다중 정의된 `reducing`, `filtering`, `mapping`, `flatmapping` `collectingAndThen` 메소드들이 있는데 대부분 프로그래머는 이를 모르고 있어도 문제가 되진 않는다. 이 수집기들은 설계 관점으로 보면 다운스트림 수집기를 작은 스트림처럼 동작하게 한 것이다.

`minBy` 와 `maxBy` 는 인수로 받은 비교자를 이용하여 스트림에서 값이 가장 작은 혹은 큰 원소를 찾아 반환한다.

`joining` 은 단순히 원소들을 연결하는 수집기를 반환하는 것에서부터 구분문자를 매개변수로 받아 문자열을 만드는 것과 접두문자와 접미문자를 받아 처리하는 문자열을 생성하는 것도 있다.
