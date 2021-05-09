# ✨ 아이템 88: `readObject` 메소드는 방어적으로 작성하라

`readObject` 는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 발생한다.

이 문제를 고치려면 `readObject` 메소드가 `defaultReadObject` 를 호출하고 나서 역직렬화된 객체가 유효한지 검사해야 한다. 검사에 실패하면 `InvalidObjectException` 을 던져 잘못된 역직렬화가 일어나는 것을 막을 수 있다.

> 역직렬화 할 때는 클라이언트가 소유해서는 안되는 객체 참조를 가지는 필드를 모두 반드시 방어적 복사해야 한다.

```java
private void readObject(OutputInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    start = new Date(start.getTime());
    end = new Date(end.getTime());

    if ( start.compareTo(end) > 0 ) {
        throw new InvalidObjectException(start + " after " + end);
    }
}
```

`readObject` 를 사용하려면 `start` 와 `end` 필드의 `final` 을 제거해야 한다.

## `readObject` 사용 여부

모든 필드 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 되는가? 아니라면 `readObject` 를 만들어 모든 유효성 검사와 방어적 복사를 수행해야한다.

## `readObject` 사용 지침

- `private` 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적 복사하라. (불변 클래스 내의 가변 요소)
- 모든 불변식을 검사하여 어긋나는 게 발견된다면 `InvalidObjectException` 을 던진다.

- 역직렬화 후 객체 그래프의 전체 유효성을 검사해야 한다면 `ObjectInputValidation` 인터페이스를 사용하라.
- 직접적이든, 간접적이든, 재정의할 수 있는 메소드는 호출하지 말자.
