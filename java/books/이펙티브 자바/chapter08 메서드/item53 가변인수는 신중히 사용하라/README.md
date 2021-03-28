# ✨ 아이템 53: 가변인수는 신중히 사용하라

[]()

가변인수 메소드는 명시한 타입의 인수를 0 개이상 받을 수 있다. 가변 인수 메소드를 호출하면 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메소드에 건네준다.

성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다.

```java
public void foo() {}
public void foo(int a) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... a4) {}
```

이렇게 다중 정의한다면 마지막 가변인수를 사용한 다중정의 메소드의 호출이 아니라면 배열의 초기화를 줄일 수 있다. EnumSet 의 정적 팩토리도 이 기업을 사용하여 열거 타입의 집합 생성 비용을 최소화한다.
