# ✨ 아이템 79: 과도한 동기화는 피하라

과도한 동기화는

1. 성능을 떨어뜨리고
2. 교착상태를 유발하고 빠뜨리며
3. 예측할 수 없는 동작을 유발한다.

> 응답 불가와 안전 실패를 피하려면 동기화 메소드나 동기화 블록 안에서 제어를 **절대로 클라이언트에 양도하면 안된다.**

## 외계인 메소드

다음은 잘못된 코드이다. 동기화 블록 안에서 외계인 메소드를 호출한다.

[ObservableSet.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/sync/ObservableSet.java)

동기화 된 영역안에서는 재정의 가능한 메소드를 호출해선 안된다. 해당 메소드가 무슨 일을 할지 알지 못하며 통제할 수 없기 때문이다. 이를 외계인 메소드라고 한다.

### 해결 방법

다행히 어렵지 않게 해결할 수 있다. 외계인 메소드 호출 동기화 블록 바깥으로 옮기면 된다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```

관찰자 리스트를 복사해쓰면 락 없이도 안전하게 순회 가능하다!

## CopyOnWriteArrayList

동기화 블록 바깥으로 옮기는 더 나은 방법도 있다. 동시성 컬렉션 라이브러리의 `CopyOnWriteArrayList` 가 이 목적으로 구현되었다.

내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되었다. 내부의 배열은 절대 수정되지 않아 순회할 때 락이 필요 없어 매우 빠르다.

> 단, 다른 용도로 사용하면 끔찍하게 느려진다.

[FixedObservableSet.java](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/sync/FixedObservableSet.java)

구현체 사용 변경으로 동기화 블록이 제거되고 앞서 본 예외 발생과 교착상태 증상이 한번에 제거되었다.

## 열린 호출

> 동기화 영역 바깥에서 호출되는 외계인 메소드를 열린 호출(open call)이라 한다.

외계인 메소드는 얼마나 오래 실행될지 알 수 없는데, 동기화 영역안에서 호출되면 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야 한다. 따라서 열린 호출은 실패 방지 효과외에 동시성 효율을 개선해준다.

## 성능적인 동기화

동기화를 초래하는 진짜 비용은 락을 얻는데 드는 CPU 시간이 아니다. 바로 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.

추가로 가상머신 코드의 최적화를 제한한다는 점도 과도한 동기화의 숨은 비용이다.

### 가변 클래스 작성

- 동기화를 전혀 하지 말고, 해당 클래스를 동시에 사용해야 하는 클래스가 외부에서 얼아서 동기화 하게 하자.
- 동기화를 내부에서 수행하여 스레드 안전한 클래스로 만들자.
  - 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등하게 개선할수 있는 경우만.

java.util 은 첫 번째 방식을 취했고(`Vector`, `HashTable` 제외), java.util.concurrent 는 두 번째 방식을 취했다.

#### 클래스 내부 동기화

- 락 분할(lock splitting)
- 락 스트라이핑(lock striping)
- 비차단 동시성 제어(nonblocking concurrency control)

등 다양한 기법을 동원하여 동시성 지원을 높히고 개선할 수 있다.

## 정리

- 교착 상태와 데이터 훼손을 피하기 위해선 **동기화 영역 안에서 외계인 메소드를 절대 호출하지 말자.**
