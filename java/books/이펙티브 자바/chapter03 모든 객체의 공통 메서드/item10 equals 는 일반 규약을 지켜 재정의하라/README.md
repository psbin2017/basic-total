# ✨ 아이템 10: equals 는 일반 규약을 지켜 재정의하라

equals 를 재정의 하지 말아야 할 상황은 다음과 같다.

- 각 인스턴스가 본질적으로 고유함: Thread 와 같은 값을 표현하는 게 아닌 동작하는 객체로 표현하는 클래스
- 논리적 동치성을 검사할 일이 없음: Pattern 와 같은 논리적 동치성을 검사하는 방법
- 상위 클래스에서 재정의한 equals 가 하위 클래스에도 부합하는 경우:
- 클래스가 private 또는 package-private 하여 equals 메서드를 호출할 일이 없는 경우

반대로 equals 를 재정의해야하는 상황은 객체가 물리적으로 같은 것이 아닌 논리적 동치성을 비교하도록 해야하는 경우이다.

- 반사성(reflexivity) : null 이 아닌 모든 참조 값 x에 대하여, `x.equals(x) == true`
- 대칭성(symmetry) : null 이 아닌 모든 참조 값 x, y에 대하여, `x.equals(y) == true` 이면 `y.equals(x) == true`
- 추이성(transitivity) : null 이 아닌 모든 참조 값 x, y, z 에 대하여 `x.equals(y) == true` 이고 `y.equals(z) == true` 이면 `x.equals(z) == true`
- 일관성(consistency) : null 이 아닌 모든 참조 값 x, y에 대하여, `x.equals(y)` 를 반복해서 호출하면 항상 true 이거나 false 를 반환
- null 이 아님 : null 이 아닌 모든 참조 값 x에 대하여, `x.equals(null) == false`

리스코프 치환 원칙: 어떤 타입에 있어 중요한 속성이라면 하위 타입도 마찬가지로 중요하다.

1. == 연산자를 사용하여 입력이 자기 자신의 참조인지 확인한다.
   - float 과 double 을 제외한 프리미티브 타입은 == 비교한다.
   - 참조 타입은 각각의 equals 메서드로 비교한다.
   - float 와 double 은 각각의 정적 메서드인 `Float.compare` 와 `Double.compare` 로 비교한다. (단, 오토박싱을 수반하여 성능에 문제가 있을 수 있다.)
   - 배열이라면 `Arrays.equals` 로 비교한다.
   - 비교하기 까다로운 필드는 해당 필드의 표준형으로 저장하여 표준형끼리 비교하면 경제적으로 비교할 수 있다.
   - 어떤 필드를 먼저 비교하느냐에 따라 성능이 차이가 있을 수 있다. 순서는 당연히 값 싼 비교부터 순서를 두어 비교한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다. (리스코프 치환 원칙)

```java
public final class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNum;

    // ...

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if ( ! ( o instanceof PhoneNumber ) ) {
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum
                && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```

복잡해 보이지만 필드들의 동치성만 검사해도 equals 의 규약을 어렵지 않게 지킬 수 있다. 또한 Object 타입 외의 매개변수를 받는 메서드는 선언하지 말자. (@Override 를 사용하면 컴파일이 되지 않고 문제를 파악하기 쉽다.)

AutoValue 프레임워크를 사용하여 비교를 수행하는 것도 좋은 방법이다.
