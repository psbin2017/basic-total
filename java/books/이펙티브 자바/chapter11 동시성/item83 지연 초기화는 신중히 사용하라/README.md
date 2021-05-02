# ✨ 아이템 83: 지연 초기화는 신중히 사용하라

지연 초기화? 필요한 시점까지 초기화를 늦추는 기법

이전 아이템과 마찬가지로 최선의 조언은 이전에 말했다. [최적화는 신중히하라](../../chapter09%20일반적인%20프로그래밍%20원칙/item67%20최적화는%20신중히하라/README.md)

> 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

지연 초기화가 초기화 순환성(initialzation circularity)을 깨뜨릴 것 같다면 `synchronized` 를 단 접근자를 사용하자.

```java
private FieldType field;

private synchronized FieldType getField() {
    if ( field == null ) {
        field = computeFieldValue();
    }
    return field;
}
```

## 홀더 관용구

> **성능 때문에 정적 필드를 지연 초기화 해야한다면 지연 초기화 홀더 클래스 관용구를 사용하자.**

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
    return FieldHolder.field;
}
```

이 관용구의 멋진점은 동기화를 전혀하지 않으니 성능이 느려질 거리가 없다는 것이다. 초기화가 끝난 후에는 VM 이 동기화 코드를 제거하여 다음부터는 아무런 검사와 동기화 없이 필드에 접근하면 된다.

## 이중 검사 관용구

> **성능 때문에 인스턴스 필드를 지연초기화 해야한다면 이중 검사 관용구를 사용하라.**

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if ( result != null ) {
        return result;
    }

    synchronized(this) {
        if (field == null) {
            field = computeFieldValue();
        }
        return field;
    }
}
```

가장 큰 궁금중은 result 변수의 필요 여부다. 해당 필드는 **이미 초기화된 상황에서는 그 필드를 딱 한번만 읽도록 보장하는 역할을 한다.**

사실 앞서본 홀더 관용구가 더 낫다.

### 단일 검사 관용구

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if ( result == null ) {
        field = result = computeFieldValue();
    }
    return result;
}
```

## 정리

대부분의 상황에서는 지연초기화가 필요 없다. 필요하다면 홀더 관용구나, 이중 검사 관용구 등을 사용하여 작성하자.
