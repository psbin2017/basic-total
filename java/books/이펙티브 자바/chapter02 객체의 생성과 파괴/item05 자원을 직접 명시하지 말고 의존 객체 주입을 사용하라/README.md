# ✨ 아이템 5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다. 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식으로 이를 해결할 수 있다.

```java
public class SpellChecker {
    private final Lexicon directory;

    public SpellChecker(Lexicon directory) {
        this.directory = Objects.requireNonNull(directory);
    }

    // ...
}
```

의존 객체 주입(의존성 주입)은 의존 객체 숫자와 의존 관계에 상관없이 동작하며 불변을 보장한다. 생성자, 정적 팩토리, 빌더 모두에 똑같이 응용할 수 있다.

이 패턴의 변형으로 생성자에 자원 팩토리를 넘겨주는 방식이 있다. 팩토리란, 팩토리 메소드 패턴으로 호출할때마다 특정 타입의 인스턴스를 반복하여 만들어주는 객체를 말한다.
