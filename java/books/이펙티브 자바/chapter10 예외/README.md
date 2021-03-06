# 💎 10장: 예외

예외를 제대로 활용한다면 프로그램의 가독성, 신뢰성, 유지보수성이 높아지지만, 잘못 사용하면 반대의 효과만 가져온다.

## 예외는 진짜 예외인 상황에서만

> 예외는 오직 예외 상황에서만 사용해야한다. 절대로 일상적인 제어 흐름용으로 사용해서는안된다.

1. 여러 스레드가 접근할 수 있다면? `Optional`, 특정 값
2. 성능이 중요하다면? `Optional`, 특정 값
3. 나머지의 경우라면? 상태 검사 메소드

## throwable

호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외. 프로그래밍 오류를 나타낼 때는 런타임 예외(runtime, Error)

## 표준 예외

| 예외 | 쓰임 |
| --- | --- |
| `IllegalArgumentException` | 허용하지 않는 값이 인수로 건네 졌을 때(null 은 따로 `NullPointerException`)
| `IllegalStateException` | 객체가 메소드를 수행하기에 적절하지 않은 상태일 때 |
| `NullPointerException` | null 을 허용하지 않는 메소드에 null 을 건넸을 때 |
| `IndexOutBoundsException` | 인덱스 범위를 넘어섰을 때 |
| `ConcurrentModificationException` | 허용하지 않는 동시 수정이 발견되었을 때 |
| `UnsupportedOperationException` | 호출한 메소드를 지원하지 않을 때 |

## 문서화

1. 검사 예외는 `@throw` 로 제공하라
2. 비검사 예외는 가능하면 제공하라
3. 검사 예외와 비검사 예외를 구분할 수 있도록 하라
4. 같은 이유로 예외를 던지면 클래스 설명에 추가 하라

## 실패 원자적

> 실패 원자적? 호출된 메소드가 실패하더라도 해당 객체는 메소드 호출 전 상태를 유지해야한다.

1. 불변 객체로 설계
2. 가변 객체인 경우 매개변수의 유효성을 검사
3. 객체의 복사본으로 작업을 수행하여 원본과 비교
4. 실패를 감지해 이전 상태로 복구

## 예외를 무시하지 말 것

만약 무시한다면 무시하는 이유를 주석으로 명시하자. 그게 아니라면 로그나 적절한 처리를 수행할 것.
