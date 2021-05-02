# ✨ 아이템 80: 스레드보다는 실행자, 태스크, 스트림을 애용하라

`java.util.concurrent` 패키지는 인터페이스 기반의 유연한 태스크 실행 기능을 담고있다.

```java
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 작업을 넘긴다.
exec.execute(runnable);

// 작업을 종료한다.
exec.shutdown();
```

이 외도 다양한 기능을 제공한다.

- 특정 태스크가 완료되기를 기다린다.
- 태스크의 모음 중 아무것 하나(`invokeAny`) 혹은 모든 태스크(`invokeAll`)가 완료되기를 기다린다.
- 실행자 서비스가 종료하기를 기다린다. (`awaitTermination`)
- 완료된 태스크들의 결과를 차례로 받는다(`ExecutorCompletionService` 이용)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다. (`ScheduledThreadPoolExecutor` 이용)

## Thread Pool Executor

큐를 둘 이상의 스레드가 처리하게 하고 싶다면 다른 정적 팩토리를 이용하여 다른 종류의 실행자 서비스를 생성하면 된다. (Thread Pool) 대부분의 실행자는 `java.util.concurrent.Executors` 의 정적 팩토리를 이용하면 된다.  `ThreadPoolExecutor` 클래스를 직접 사용해서 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.

가벼운 프로그램/서버이라면 `Executor.newCachedThread` 이 일반적으로 좋은 선택이다.

프로덕션 서버라면 `Executor.newFixedThreadPool` 을 선택하거나 완전히 통제 가능한 `ThreadPoolExecutor` 가 좋다.

> 손수 작업 큐를 만드는 경우는 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다.

스레드를 직접 다루게 되면 Thread 가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다.

반면 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다.

- 작업 단위: 태스크
  - Runnable : 실행만 한다.
  - Callable : 값을 반환하고 예외를 던질수 있다.

- 태스크를 수행하는 실행 메커니즘 : 실행자 서비스

태스크 수행을 실행자 서비스에게 맡기면 원하는 태스크 수행 정책을 선택가능하고, 요건이 바뀌면 언제든지 변경가능하다.

## Java 7

포크-조인(fork-join) 태스크를 지원하도록 확장되었다.

포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다.

`ForkJoinTask` 의 인스턴스는 작은 하위 태스크로 나뉘어질 수도 있고, `ForkJoinPool` 을 구성하는 스레드의 남은 태스크를 가져와서 대신 처리할 수도 있다.

이렇게 하여 CPU 자원을 최대한 활용하여 높은 처리량과 낮은 지연시간을 달성할 수 있다.

직접 작성하고 튜닝하기란 매우 어려운 일이지만, 포크-조인 풀을 이용하여 만든 병렬 스트림을 사용한다면 적은 노력으로도 해당 이점을 사용할 수 있다. [스트림 병렬화는 주의해서 적용하라](../../chapter07%20람다와%20스트림/item48%20스트림%20병렬화는%20주의해서%20적용하라/README.md)
