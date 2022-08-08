# 구조

- 개발자가 개발한 `Source Code` (.java) 를 Java Compiler 를 통해 변환
- 컴파일된 `Byte Code` (.class) 를 JVM `Class Loader` 로 전송한다.
- `Class Loader` 는 클래스 파일을 불러와 메모리에 저장한다.
- JVM `Execution Engine` 은 `Class Loader` 에 저장된 `Byte Code` 를 명령어 단위로 나누어 실행한다.
- JVM `Garbage Collector` 는 필요로 하지 않는 객체를 메모리에서 수거한다.
- Runtime Data Area 는 JVM 이 운영체제에개 할당받은 메모리 공간으로 `Class Loader`, `Execution Engine`, `Garbage Collector` 이 이에 속한다.

## Runtime Data Area

| 영역 | 정보 | 스레드 |
| --- | --- | --- |
| Method Area | 클래스, 전역 변수, 상수 | 모든 스레드 |
| Heap Area | new 객체, 배열, GC 의 수집 영역 | 모든 스레드 |
| Stack Area | 지역 변수, 메소드 매개변수 | 개별 스레드 |
| PC Register | JVM 이 실행되는 위치(어떤 부분을 어느 명령어 실행하는지 주소) | 개별 스레드 |
| Native Method Stack | 네이티브 메소드를 만나면 저장되는 영역 | 개별 스레드 |

### Method Area

> 클래스 정보, 전역 변수, 상수, 모든 스레드가 공유하는 영역

- JVM 이 실행되면서 생긴다.
- Class 정보, 전역 변수, Static 변수가 저장된다.
- Runtime Constant Pool : 상수가 저장된다.
- 모든 스레드에 정보가 공유된다.

### Heap Area

> new 객체(배열), GC 의 수거 영역, 모든 스레드가 공유하는 영역

- new 연산자로 생성된 객체. 배열도 여기에 해당한다.
- 메소드 영역에서 로드된 클래스가 여기에 생성된다.
- `Garbage Collector` 가 참조가 해제된 객체를 확인하고 수거하는 영역이다.

### Stack Area

> 지역 변수, 매개 변수

- 지역 변수, 메소드의 매개변수와 같이 잠깐 사용되고 필요 없는 데이터가 저장된다.
- LIFO
- 지역 변수이지만 참조 타입을 가진 객체라면 Heap 에 저장된 주소값을 Stack 에 저장해서 사용한다.
- 스레드마다 하나씩 가지고 있다.

### PC Register

> JVM 실행 위치, 각 스레드가 가지는 영역

- 스레드가 생성되면 생기는 영역이다.
- 스레드가 어떤 부분을 무슨 명령어로 실행해야할 지 기록한다.
- JVM 이 실행하고 있는 현재 위치를 저장한다.

### Native Method Stack

> JNI 메소드 실행 공간

- Java 가 아닌 다른 언어로 실행해야하는 메소드를 실행할 때 사용하는 공간

## 참고 내용

- [JVM 메모리 구조란? (JAVA)](https://steady-coding.tistory.com/305)
- [자바의 메모리 구조](https://velog.io/@shin_stealer/%EC%9E%90%EB%B0%94%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B5%AC%EC%A1%B0)
- [JVM 구조와 메모리 영역 - Method, Heap, Stack Area](https://tape22.tistory.com/28)
- [스레드와 메모리구조](https://thsd-stjd.tistory.com/149)
