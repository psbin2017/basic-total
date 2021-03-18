# ✨ 아이템 36: 비트 필드 대신 EnumSet 을 사용하라

비트 필드를 사용하면 비트별 연산을 사용하여 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다. 그러나 비트 필드는 정수 열거 상수의 단점을 그대로 가지며, 다음의 문제점을 추가로 가진다.

- 비트 필드 값이 그대로 출력시 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 까다롭고 어렵다.
- 비트 필드에 녹아있는 모든 원소를 순회하기도 까다롭다.
- 최대 몇 비트가 필요한지 API 작성시 미리 예측하여 적절한 타입을 선택해야 한다. (API 를 수정하지 않는 이상 비트수를 더 늘릴 수 없다.)

EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다. 원소가 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

```java

public class Text {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHROUGH;
    }

    public void applyStyles(Set<Style> styles) {
        // ...
    }
}

// text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

EnumSet 을 건네는 상황이라도 Set 인터페이스로 받는게 일반적으로 좋은 습관이다.

## 정리

결론적으로 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. EnumSet 클래스가 비트필드 수준의 명료함과 성능을 제공한다. 단점으로 불변을 만들수 없다는 점인데 이는 성능을 감수하고 `Collections.unmodifiableSet()` 으로 감싸 사용할 수 있다.

