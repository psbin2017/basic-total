# ✨ 아이템 35: ordinal 메소드 대신 인스턴스 필드를 사용하라

ordinal 은 열거 타입에서 몇번째 위치인지 반환하는 메소드이다. 시스템에서 주어지는 값이기에 이를 활용하는 방법도 있겠지만 이는 위험한 방법이다.

그러나 이 메소드를 쓸 일이 없다. 이 메소드는 EnumSet 과 EnumMap 과 같이 열거 타입 기반의 범용 자료구조 목적으로 설계되었다. 따라서 이런 용도가 아니라면 다음과 같이 명시적으로 인스턴스 필드를 사용하는 것이 바람직하다.

[인스턴스 필드 사용 예제](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/enums/Ensemble.java)
