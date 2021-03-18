# ✨ 아이템 29: 이왕이면 제네릭 타입으로 만들라

JDK 가 제공하는 제네릭 타입과 메서드를 사용하는 일은 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다.

## 이전 예제

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Obeject[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0 ) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 참조해제 (item7)
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Array.copyOf(elements, 2 * size + 1);
        }
    }
}
```

## 제네릭 타입

1. 타입 매개변수를 추가한다.

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        // point 1.
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0 ) {
            throw new EmptyStackException();
        }
        // point 2.
        E result = elements[--size];
        elements[size] = null; // 참조해제 (item7)
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Array.copyOf(elements, 2 * size + 1);
        }
    }
}
```

첫 번째 방법. 제네릭 배열 생성 금지를 우회하는 방법

```java
// 배열 elements 는 push(E) 로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안정성을 보장하지만, 이 배열의 타입은 E[] 가 아닌 Object[] 다.
@SuppressWarnings("unchecked")
elements = (E[]) new E[DEFAULT_INITIAL_CAPACITY];
```

두 번째 방법. elements 필드의 타입을 E[] 에서 Object[] 로 바꾸는 방법

```java
// push 에서 E 타입만 허용하므로 이 형변환은 안전하다.
@SuppressWarnings("unchecked")
E result = (E) elements[--size];
```

첫 번째 방식은 형변환을 배열 생성시 한번만 해주면 되지만, 두 번째 방식은 배열에서 원소를 읽을 때 마다 해주어야 한다. 일반적인 경우라면 첫 번째가 선호되지만 첫 번째 방법은 배열의 **런타임타입이 컴파일타입과 달라 힙 오염을 일으킨다.** (다만 이번 예제에서는 힙 오염이 해가 되지 않았다.)

타입 매개변수에 제약을 두는 제네릭 타입 또한 있다. (java.util.concurrent.DelayQueue) ... `DelayQueue<E extends Delayed> ...` 이 경우 자신을 사용하는 클라이언트는 DelayQueue 원소에서 형변환 없이 곧바로 Delayed 클래스의 메소드를 호출할 수 있다. 이러한 타입을 타입 매개변수 E 를 한정적 타입 매개변수라고 한다.

## 정리

클라이언트에서 직접 형변환해야하는 타입보다는 제네릭 타입이 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계한다면 형변환 없이도 사용할 수 있도록 하라.

기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 클라이언트에는 영향을 주지않으면서 새로운 사용자를 훨씬 편하게 해주는 길이다.
