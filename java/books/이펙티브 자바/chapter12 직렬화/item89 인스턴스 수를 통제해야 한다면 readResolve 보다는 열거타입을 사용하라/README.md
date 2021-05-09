# ✨ 아이템 89: 인스턴스 수를 통제해야 한다면 `readResolve` 보다는 열거타입을 사용하라

> `readResolve` 인스턴스를 통제 목적으로 사용해야한다면 객체 참조 타입 인스턴스 필드는 모두 `transient` 로 선언해야 한다.

```java
public enum Elvis {
    INSTANCE;

    private String[] favoriteSongs = { "Hound Dog", "Heartbreak Hotel" };

    public void printFavorites() {
        // ...
    }
}
```

`readResolve` 는 접근성이 중요하다. `final` 클래스라면 `readResolve` 는 `private` 이어야 한다.

`final` 이 아니라면 몇 가지를 주의해야 한다.

1. `private` 로 선언하면 하위 클래스에서 사용할 수 없다.
2. `package-private` 로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.
3. `protected` 나 `public` 을 선언하면 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.
4. `protected` 나 `public` 이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 `ClassCaseException` 을 일으킬 수 있다.

## 정리

불변식을 지키기 위해서 인스턴스를 통제해야 한다면 가능한 한 열거타입을 사용하자. 여의치 않은 상황에서 직렬화와 인스턴스 통제가 필요하다면 `readResolve` 메소드를 작성해야 하며, 해당 클래스의 모든 참조 타입 인스턴스 필드를 `transient` 를 선언해야 한다.
