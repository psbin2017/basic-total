# ✨ 아이템 72: 표준 예외를 사용하라

예외도 재사용하는 것이 좋으며 자바 라이브러리는 대부분 API 에서 쓰기에 충분한 수의 예외를 제공한다.

표준 예외를 재사용하면 얻는게 많다.

1. API 가 다른 사람이 익히기 쉬운 코드가 되고
2. 낮선 예외를 사용하지 않아 읽기 쉬우며
3. 새로 클래스를 만들지 않아 클래스를 적재하는 시간도 적게 걸린다.

| 예외 | 쓰임 |
| --- | --- |
| `IllegalArgumentException` | 허용하지 않는 값이 인수로 건네 졌을 때(null 은 따로 `NullPointerException`)
| `IllegalStateException` | 객체가 메소드를 수행하기에 적절하지 않은 상태일 때 |
| `NullPointerException` | null 을 허용하지 않는 메소드에 null 을 건넸을 때 |
| `IndexOutBoundsException` | 인덱스 범위를 넘어섰을 때 |
| `ConcurrentModificationException` | 허용하지 않는 동시 수정이 발견되었을 때 |
| `UnsupportedOperationException` | 호출한 메소드를 지원하지 않을 때 |

복소수, 유리수를 다루는 객체라면 `ArithmeticException`, `NumberFormatException` 을 재사용할 수 있을 것이다.

상황에 부합하면 표준 예외를 사용하는 것이 좋고, 더 많은 정보를 주길 원한다면 표준 예외를 확장해도 좋다. 예외는 직렬화할 수 있다. 직렬화에는 부담이 있으니 사실만으로도 나만의 예외를 새로 만들지 않아야 한다.

인수의 값이 무엇이었든 어차피 실패했을 꺼라면 `IllegalStateException` 를 그렇지 않으면 `IllegalArgumentException` 를 던지자.