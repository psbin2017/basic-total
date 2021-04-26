# 단어 사전

## 💎 2장 객체의 생성과 파괴

## 💎 6장: 열거 타입과 애너테이션

비트 필드:

## 💎 7장 람다와 스트림

플루언트 API: [플루언트 인터페이스](https://ko.wikipedia.org/wiki/%ED%94%8C%EB%A3%A8%EC%96%B8%ED%8A%B8_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4). 대표적인 예제는 스트림 API 와 빌더 패턴이 될 수 있다.

### 참조 지역성(Locality Of Reference)

동일한 값 또는 해당 값에 관계된 스토리지 위치가 자주 액세스되는 특성.

- 공간(spatial) 지역성
  - 특성 클러스터의 기억 장소들에 대해 참조가 집중적으로 이루어지는 경향
  - 참조된 메모리 근처의 메모리를 참조
- 시간(temporal) 지역성
  - 최근 사용되었던 기억 장소들이 집중적으로 액세스되는 경향
  - 참조했던 메모리는 빠른 시간에 다시 참조될 확률이 높음
- 순차(sequential) 지역성
  - 데이터가 순차적으로 액세스되는 경향
  - 프로그램 내의 명령어가 순차적 구성에 기인

1. [출처 1](https://itwiki.kr/w/%EC%B0%B8%EC%A1%B0_%EC%A7%80%EC%97%AD%EC%84%B1)
2. [출처 2](https://en.wikipedia.org/wiki/Locality_of_reference)

## 💎 11장: 동시성

원자성: 어떤 것이 더 이상 쪼개질 수 없는 성질을 말한다.

안전 실패(safety failure): 프로그램이 잘못된 결과를 계산할 경우를 안전 실패라고 한다.

안전 발행(safety publication): 객체에서 공유하는 부분만 동기화를 진행하며, 다른 스레드에서 동기화 없이 자유롭게 값을 읽을 수 있는 것. 해당 객체는 불변(effectively immutable)으로 볼 수 있다.
## 디자인 패턴

- 플라이웨이트(`Flyweight`)
- 정적 팩토리 메소드(`Stataic Factory Method`)
- 빌더(`Builder`)
- 싱글톤(`Singleton`) ... LazyHolder
- 어답터(`Adopter`)
- 타입 안전 이종 컨테이너(`Heterogeneous Container`)
