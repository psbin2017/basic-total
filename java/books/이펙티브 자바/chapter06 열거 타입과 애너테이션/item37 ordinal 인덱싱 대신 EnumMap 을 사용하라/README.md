# ✨ 아이템 37: ordinal 인덱싱 대신 EnumMap 을 사용하라

EnumMap 의 장점

- 배열과 비교하여 안전하지 않은 형변환을 사용하지 않는다.
- 배열과 비교하여 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
```

## 스트림 사용 방법

```java
Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet() )));
```

스트림 사용시 명시적으로 EnumMap 구현체를 사용하여 만든다. 둘의 차이점은 EnumMap 은 항상 Enum 개수 만큼 맵을 만들고 스트림 버전에서는 존재하는 Enum 개수 만큼 맵을 만든다.

## 예제

[Phase.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/Phase.java)

```java
public enum Phase {
    // ...
    PLASMA;

    public enum Transition {
        // ...
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
    }
}
```

새로운 열거타입이 추가되어도 나머지 코드를 수정할 필요가 없다. 맵들이 맵이 배열들로 구현되어 낭비되는 공간과 시간도 거의 없어 명확하고 유지보수가 좋다.

## 추가 참고내용

- [Sup2is 님 : Java 자료구조 파헤치기 #12 EnumMap](https://sup2is.github.io/2019/11/11/enum-map.html)
