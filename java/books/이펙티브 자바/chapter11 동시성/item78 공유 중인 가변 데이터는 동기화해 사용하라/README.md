# ✨ 아이템 78: 공유 중인 가변 데이터는 동기화해 사용하라

`synchronized` 키워드는 메소드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이니 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하도록 막는 용도로만 생각한다.

1. 한 객체가 일관 된 상태를 가지고 생성 된다. [item 17](../../chapter04%20클래스와%20인터페이스/item17%20변경%20가능성을%20최소화하라/README.md)
2. 생성된 객체에 접근하는 메소드는 객체에 락(lock)을 건다.
3. 락을 건 메소드는 객체의 상태를 확인하고 필요 시 수정한다.

해당 결과로는 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. **동기화를 제대로 사용하면 어떤 메소드도 해당 객체의 상태가 일관되지 않은 순간을 볼 수 없다.**

그러나 동기화에는 중요한 기능이 하나 더 있다. **동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.**

> 동기화는 일관성이 깨진 상태를 볼 수 없으며, 동기화된 메소드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

1. (A Thread) → [(B Method/Block)] → [(B Method/Block A Thread changed)]
2. (C Thread) → **[(B Method/Block A Thread changed)]** → ...

## 원자성

> 원자성(原子性, atomicity)은 어떤 것이 더 이상 쪼개질 수 없는 성질을 말한다. [위키백과](https://ko.wikipedia.org/wiki/%EC%9B%90%EC%9E%90%EC%84%B1)

어떠한 작업이 실행된다고 가정,

- 언제나 완전하게 진행되어 종료되거나,
- 그럴 수 없는 경우 실행을 하지 않는 경우를 말한다.

원자성을 가지는 작업은 실행되어 진행되다가 종료하지 않고 **중간에서 멈추는 경우는 있을 수 없다.**

> 자바 언어 명세상 long 과 double 외의 변수를 읽고 쓰는 동작은 원자적이다. 다수의 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다.

## 그래서?

자바 언어 명세는 스레드가 필드를 읽을 때 **항상 '수정이 완전히 반영된' 값을 얻는다고 보장**하지만, 한 스레드가 **저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.**

> **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.**

한 스레드가 만든 변화가 다른 스레드에 언제 어떻게 보이는지를 규정한 **자바의 메모리 모델** 때문이다.

## 예제

공유 중인 가변 데이터를 원자적으로 읽고 사용할 수 있더라도 동기화에 실패하면 처참한 결과로 이어진다. (`Thread.stop` 쓰지 말자, 11 에서 제거되었다.)

동기화를 하지 않으면 **메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤 보게 될지 보장할 수 없다.**

동기화가 빠지면 가상 머신이 다음과 같은 최적화를 수행할 수도 있는 것이다.

```java
// 원래 코드
while (!stopRequested) {
    i++;
}

// 최적화 한 코드
if (!stopRequested) {
    while (true) {
        i++;
    }
}
```

실제로 적용하는 끌어올리기(hoisting)라는 최적화 기법이다. 이 결과 응답 불가(liveness failure) 상태가 되어 더이상 진전이 없다.

## `StopThread`

[StopThread](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/sync/StopThread.java)

> 쓰기와 읽기 모두 동기화되지 않으면 동작을 보장하지 않는다.

1. 읽기와 쓰기를 모두 동기화 (`synchronized`)
2. 가장 최근에 기록된 값을 읽게 됨을 보장 (`volatile`) ... 단, 배타적 수행과는 상관 없다.

`volatile` 는 주의해서 사용해야 한다.

## 안전 실패 (safety failure)

```java
private static volatile int nextSerialNumber = 0;

// 따라하지 말 것 !!!!!!
public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

`++` 연산자가 문제인데, 코드상으로는 하나지만 실제로는 `nextSerialNumber` 필드에 두 번 접근한다. 값을 읽고, 다음에 값을 1 증가시킨 값을 저장한다.

만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다. **프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패(safety failure) 라고 한다.**

```java
private static int nextSerialNumber = 0;

// 추가로 최댓값 도달시 예외를 던지면 완성
public static synchronized int generateSerialNumber() {
    return nextSerialNumber++;
}
```

더 좋은 해결 방법이 있다. `java.util.concurrent.atomic` 패키지의 `AtomicLong` 을 사용하자. 이 패키지에는 락 없이도 (lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. 이 패키지는 배타적 수행까지 지원한다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

## Best Case

사실 제일 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 불변 데이터만 공유하거나 공유하지 말자.

> **가변 데이터는 단일 스레드에서만 쓰도록 노력하자.**

이 정책이 받아들여지면 이 사실을 문서에 남기고 유지보수 과정에서도 정책이 지켜지도록 하는 것이 중요하다.

## 안전 발행

한 스레드가 데이터를 다 수정하고 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화를 진행해도 된다. 그러면 해당 객체를 다시 수정할 일이 생기기 전까진, **다른 스레드들이 동기화 없이 자유롭게 값을 읽어 나갈 수 있다.**  이런 객체는 사실상 불변(effectively immutable)이라 하고 다른 스레드에서 이런 객체를 건네는 행위를 안전 발행(safe publication)이라고 한다.

## 정리

객체를 안전하게 발행하는 방법은 많다.

클래스 초기화 과정에서 객체를

1. 정적 필드
2. volatile 필드
3. final 필드
4. 보통의 락을 통한 접근 하는 필드
5. 동시성 컬렉션에 저장

> **여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다.**

스레드 끼리의 통신만 필요하다면 `volatile` 한정자만으로 동기화 할 수 있으나, 올바르게 사용하는게 까다롭다.
