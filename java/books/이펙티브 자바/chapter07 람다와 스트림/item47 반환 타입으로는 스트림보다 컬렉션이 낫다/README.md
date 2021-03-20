# ✨ 아이템 47: 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀀스를 반환하는 공개 AI 의 반환 타입에는 Collection 이나 그 하위 타입을 쓰는 게 일반적으로 최선이다. (Arrays 역시 `Arrays.asList` 와 `Stream.of` 쉽게 반복 스트림을 제공할 수 있다.)

시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList 나 HashSet 같은 표준 컬렉션 구현체로 반환하는게 최선일 수 도있다. 그러나 단지 컬렉션을 반환한다는 이유로 덩치가 큰 시퀀스를 메모리에 올려선 안된다.

[Power Set](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/datastructure/PowerSet.java)

[Sub List](https://github.com/psbin2017/garbage-collection/blob/master/gc/src/test/java/com/collection/gc/sample/datastructure/SubList.java)

컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라. 만약 Stream 인터페이스가 Iterable 을 지원하도록 수정된다면, 그때는 안심하고 Stream 을 반환하면 될 것이다. (스트림 처리와 반복을 모두 사용할 수 있으니).
