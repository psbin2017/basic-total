# ✨ 아이템 90: 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

> 직렬화 프록시 패턴(serialization proxy pattern)은 버그와 보안 문제를 크게 줄여준다.

별로 복잡하지도 않다.

```java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID = 1247120571027509L; // 생성된 값
}
```

바깥 클래스에 직렬화 프록시 패턴용 `writeReplace` 메소드를 작성한다.

```java
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

`writeReplace` 로 직렬화 시스템은 결코 바깥 클래스의 직렬화된 인스턴스를 생성할 수 없게 된다. 다음으로 `readObject` 를 추가한다.

```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

마지막으로 바깥 클래스와 논리적으로 동일한 `readResolve` 메소드를 `SerializationProxy` 에 추가한다.

```java
private Object readResolve() {
    return new Period(start, end); // public 생성자
}
```

직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다. 한계도 명확하다. 클라이언트가 멋대로 확장할 수 있는 클래스는 적용할 수 없다. 객체 그래프의 순환이 있는 클래스에도 적용할 수 없다. 방어적 복사 때보다 느려진다.
