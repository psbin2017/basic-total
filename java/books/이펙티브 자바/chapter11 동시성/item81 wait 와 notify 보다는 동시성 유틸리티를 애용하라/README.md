# ✨ 아이템 81: wait 와 notify 보다는 동시성 유틸리티를 애용하라

> wait 와 notify 는 올바르게 사용하기 까다로우니 고수준 동시성 유틸리티를 사용하자

`java.util.concurrent` 는 크게 세 범주로 나눌 수 있다.

- 실행자 프레임워크 (item80)
- 동시성 컬렉션 (concurrent collection)
- 동기화 장치 (synchronizer)

## 동시성 컬렉션

> `List`, `Queue`, `Map` 같은 표준 컬렉션 인터페이스로 하겠습니다. 근데 이제 동시성을 곁들인 ..

동시성 컬렉션에서 동시성을 무력화하는 건 불가능하고, 외부에서 락을 추가로 쓰면 속도가 느려진다.

여러 메소드를 원자적으로 묶어 호출하는 것도 불가능한데, 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메소드들이 수정되었다.

### ConcurrentHashMap

`Map` 의 `putIfAbsent(key, value)` 는 주어진 키에 매핑된 값이 아예 없을 때만 새 값을 집어넣는다. 기존 값이 있으면 해당 값을 반환하고, 없다면 null 을 반환한다.

```java
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null) {
            result = s;
        }
    }
    return result;
}
```

`ConcurrentHashMap` 은 동시성이 뛰어나며 속도도 무척 빠르다.

### BlockingQueue

`Queue` 를 확장한 `BlockingQueue` 는 작업이 성공적으로 완료될 때까지 기다리도록 하는 메소드가 추가되었다. 이런 특성 덕에 `BlockingQueue` 는 생산자-소비자 큐로 쓰기 적합하다.

- 하나 이상의 생산자가 큐에 작업을 추가하고,
- 하나 이상의 소비자가 큐에 있는 작업을 사용한다.

> `ThreadPoolExecutor` 를 포함한 대부분의 실행자 서비스 구현체에서 이 `BlockingQueue` 를 사용한다.

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.

## 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.

가장 자주 쓰이는 동기화 장치는 `CountDownLatch` 와 `Semaphore` 이다. 다음으로는 `CyclicBarrier` 와 `Exchanger` 이다. 가장 강력한 동기화 장치는 `Phaser` 이다.

### CountDownLatch

`CountDownLatch` 는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. 유일한 int 생성자로 해당 값이 `countDown` 메소드를 몇 번 호출해야 대기중인 스레드들을 깨우는지 결정한다.

어떤 동작을 동시에 시작하여 완료하기까지의 시간을 재는 기능을 구현한다면,

- 동작을 실행하는 실행자
- 동시에 수행할 수 있는지를 뜻하는 동시성 수준
- 타이머 스레드

작업자 스레드가 준비를 마치면 타이머 스레드가 시작을 하게하고 마지막 스레드가 동작을 마치면 타이머를 종료한다.

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurreny);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for ( int i = 0; i < concurreny; i++ ) {
        executor.execute(() -> {
            // 타이머에게 준비를 마쳤음을 알림
            ready.countDown();
            try {
                // 모든 작업자 스레드가 준비될 때까지 대기
                start.await();
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                // 타이머에게 작업을 마쳤음을 알림
                done.countDown();
            }
        })
    }

    // 모든 작업자가 준비될 때까지 대기
    ready.await();
    long startNanos = System.nanoTime();
    // 작업자들을 깨운다.
    start.countDown();
    // 모든 작업자들이 작업을 마칠 때까지 대기
    done.await();
    return System.nanoTime() - startNanos;
}
```

주의사항으로 `concurrency` 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 해당 메소드는 결코 끝나지 않는다. (스레드 기아 교착상태:thread starvation deadlock)

덤, 시간 간격을 잴 때는 항상 `System.currentTimeMillis` 가 아닌 `System.nanoTimes` 를 사용하자.

## 레거시

새로운 코드라면 동시성 유틸리티를 사용해야 하겠지만, 레거시 코드는 어쩔 수 없이 `wait` 와 `notify` 를 다루어야한다.

> wait 메소드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대 호출하지 말자.

```java
synchronized (obj) {
    while (<조건이 충족되지 않음>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 락을 잡는다.
    }
    // 조건이 완료되면 동작을 수행한다.
}
```

조건이 만족되지 않아도 스레드가 깨어나는 상황은 다음과 같다.

- 스레드가 `notify` 를 호출한 다음 다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 `notify` 를 호출한다. 공개 객체를 락으로 사용해 대기하는 클래스는 이러한 위험에 노출되어 있다.
- 깨우는 스레드는 지나치게 관대하여, 대기 중인 스레드 중 일부만 조건이 충족되어도 `notifyAll` 을 호출하여 모든 스레드를 깨울 수 있다.
- 드물게 `notify` 없이 깨어난다. (허위 각성: spurious wakeup)

> 일반적으로는 `notify` (하나의 스레드만 깨움) 보다 `notifyAll` (모든 스레드를 깨움) 을 사용하는게 합리적이고 안전한 조언이 된다.

모든 스레드가 같은 조건을 기다리고, 조건이 충족될 때마다 단 하나의 스레드만 혜택을 받는다면 `notify` 를 사용해 최적화 할 수 있다.

그럼에도 불구하고 `notifyAll` 를 사용해야하는 이유는 외부 공개 객체에 대해 실수 혹은 악의적으로 `notify` 를 호출하는 상황에 대비하기 위해 `notifyAll` 를 사용하는게 좋다.

## 정리

- 새로운 코드라면 `java.util.concurrent` 에서 제공하는 내용을 활용하여 동시성 프로그래밍을 슬기롭게 헤쳐나가자.
- 레거시라면 `wait` 는 반드시 대기 반복문 관용구로 작성하고, `notify` 보다는 `notifyAll` 을 사용하자.
