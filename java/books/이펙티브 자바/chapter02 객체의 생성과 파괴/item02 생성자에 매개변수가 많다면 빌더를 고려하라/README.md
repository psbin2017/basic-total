# ✨ 아이템 2: 생성자에 매개변수가 많다면 빌더를 고려하라

앞서본 정적 팩토리 메소드를 사용하는 방법도 있지만, 매개변수가 많아지면 클라이언트 코드를 작성하고 읽는 것이 어려워진다.

자바빈즈 패턴은 객체하나를 만들기 위해서 여러번의 메소드를 호출해야 하고, 완성이 되기 전까지 일관성이 무너진 상태에 놓이게 된다.

빌더 패턴은 명명된 선택적 매개변수(named optional parameters)를 흉내 낸것으로 계층적으로 설계된 클래스와 함께 사용하기 좋다. 이 패턴을 쉽게 만들어 주는 것이 롬복의 `@Builder` 이다.

```java
@Getter
public class Parent {
    public String name;
    public int score;

    @Builder
    public Parent(String name, int score) {
        this.name = name;
        this.score = score;
    }
}

@Getter
public class Child extends Parent {
    public String childName;
    public int childScore;

    // @Builder
    @Builder(builderMethodName = "childBuilder")
    public Child(String name, int score, String childName, int childScore) {
        super(name, score);
        this.childName = childName;
        this.childScore = childScore;
    }
}
```

상속 관계를 가진 빌더 사용시 컴파일 에러를 막기 위해 자식 빌더 객체에 빌더 메소드를 새로 명명해야한다. 다른 방법으로 `@SuperBuilder` 를 클래스에 선언하여 사용하는 방법이 있으며 이 경우 `@Builder` 를 사용할 수 없다.
