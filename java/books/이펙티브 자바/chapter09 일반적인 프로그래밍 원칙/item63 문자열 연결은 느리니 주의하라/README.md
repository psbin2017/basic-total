# ✨ 아이템 63: 문자열 연결은 느리니 주의하라

문자열 연결 연산자로 문자열 n 개를 잇는 시간은 n 제곱에 비례한다.

```java
String result = "";
for (int i = 0; i < numItems(); i++) {
    result += lineForItem(i);
}
```

문자열은 불변이기 때문에 두 문자열을 연결하는 경우 양쪽 내용을 모두 복사해야하므로 성능 저하를 피할 수 없다.

성능을 포기하고 싶지 않다면 String 대신 StringBuilder 를 사용하자.
