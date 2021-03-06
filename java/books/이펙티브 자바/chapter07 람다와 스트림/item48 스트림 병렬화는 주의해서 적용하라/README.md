# ✨ 아이템 48: 스트림 병렬화는 주의해서 적용하라

- 초창기 릴리스는 wait/notify
- 자바 5 부터는 동시성 컬렉션인 `java.util.concurrent`
- 자바 7 부터는 고성능 병렬분해 (parallel decom-position) 프레임워크인 포크-조인  (fork-join) 패키지
- 자바 8 부터는 parallel 메서드 호출만으로 파이프라인 병렬 실행할 수 있는 스트림을 지원

자바로 동시성 프로그램을 작성하기 점점 쉬워지고는 있지만 올바르고 빠르게 작성하는 일은 여전히 어렵다.

좋은 성능을 가지더라도 **데이터 소스가 Stream.iterate 이거나 중간 연산으로 limit 를 사용하면 파이프라인 병렬화를 기대할 수 없다.**

파이프라인 병렬화가 작업을 CPU 코어 수 만큼 병렬로 수행한다고 해보자. 쿼드코어 기준 19 번째 계산까지 마치고 마지막 20 번째 계산이 수행하는 시점에는 나머지 3 개의 코어는 일을 하지 않는다. 따라서 21, 22, 23 번째 기능은 20 번째 계산이 끝나더라도 이 계산은 끝나지 않는다. 각 20 번째 보다 2배, 4배, 8배의 시간이 소요된다.

쉽게 정리하면 **스트림 파이프라인을 마구잡이로 병렬화 해선 안된다.** 병렬화시 오히려 성능이 더 나빠질 수도 있다.

대체로 스트림 소스가 `ArrayList`, `HashMap`, `HashSet`, `ConcurrentHashMap` 의 인스턴스거나 배열 int 범위, long 범위일 때 병렬화의 효과가 가장 좋다. 해당 자료구조는 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 일을 다수의 스레드에 분배하기에 좋다는 특징을 가지고 있다. 분할하는 작업은 `Spliterator` 가 담당하며 `Spliterator` 객체는 `Stream` 이나 `Iterable` 의 `spliterator()` 메소드로 얻어올 수 있다.

## 참조 지역성

이 자료구조의 다른 공통점은 원소들을 순차적으로 실행할 때 참조 지역성(locality of reference)이 뛰어나다는 것이다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어있다는 뜻이다. 그러나 참조가 가리키는 실제 객체가 메모리에서 서로 떨어져 있을 수 있는데, 이렇게 되면 참조 지역성이 나빠진다.

참조 지역성이 낮으면 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분의 시간을 기다리게 되어 오버헤드가 발생한다. 따라서 **참조 지역성은 대량의 데이터를 처리하는 벌크 연산을 병렬화할 때  아주 중요한 요소로 작용하게 된다.** 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다. 기본 타입 배열은 데이터 자체가 메모리에 연속해서 저장되기 때문이다. (참조가 아니다.)

## 종단 연산

종단 연산의 동작 방식은 병렬 수행 효율에 영향을 준다. 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 큰 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행 효과는 제한될 수 밖에 없다. **종단 연산의 가장 적합한 작업은 축소(reduction)이다.**

축소는 파이프라인으로 만들어진 모든 원소를 하나로 합치는 작업으로 `Stream.reduce` 메소드 중 `min`, `max`, `count`, `sum` 과 같은 완성된 형태로 제공되는 메소드 중 하나를 수행한다.

`anyMatch`, `allMatch`, `nonMatch` 처럼 조건에 맞으면 바로 반환되는 메서드도 적합하다. 반면, 가변 축소(mutable reduction)를 수행하는 Stream 의 collection 메소드는 병렬화에 적합하지 않다. 컬렉션을 합치는 부담이 크기 때문이다.

**스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상하지 못한 동작 에러가 발생할 수 있다.**

결과가 잘못되거나 오동작하는 것을 안전 실패라고 한다. 스트림에서 안전 실패는 병렬화된 파이프라인이 사용하는 `mappers`, `filters` 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 발생할 수 있다.

Stream 에서 명세는 함수 객체에 관한 엄중한 규약을 정의해놓았다. 예를 들어 reduce 연산으로 건네지는 누적기와 결합기 함수는 반드시 결합법칙을 만족하고 간섭받지 않고 상태를 갖지 않아야 한다.

결합 법칙 만족 조건

> (a op b) op c == a op (b op c)

무상태성

> 파이프라인이 수행되는 동안 데이터 소스는 변경되지 않아야 한다.

## 병렬 파이프라인 수단

병렬이 아닌 순차 수행의 경우 올바른 경우를 얻을 수도 있다. 그러나 병렬로 수행하는 경우 문제가 발생한다. 스트림의 병렬화는 오직 성능 최적화의 수단임을 기억해야한다.

성능 향상을 추정하는 방법은 스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱한다. 이 값이 최소 수십만은 되어야 성능 향상을 눈에 띄게 가져올 수 있다.

**조건만 된다면 parallel 메소드 호출만으로 프로세서 코어 수에 비례하는 성능향상을 가져올 수도 있다.**

[ParallelCaseSample](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/stream/ParallelCaseSample.java)

다만 해당 스트림의 문제라면 n 이 커질수록 느려진다. (양쪽다) 무작위의 수들로 이루어진 스트림을 병렬화 하는 경우 `ThreadLocalRandom` 보다는 `SplittableRandom` 인스턴스를 사용하는 것이 좋다. 위 케이스의 경우 병렬화하면 성능이 선형으로 증가한다. 반면 `ThreadLocalRandom` 는 싱글 스레드에 사용되도록 만들어졌다. 병렬용 스트림으로 사용가능하지만 `SplittableRandom` 만큼 빠를 수 없다. 마지막으로 `Random` 은 모든 연산을 동기화하므로 병렬 처리시 최악의 성능을 가진다.
