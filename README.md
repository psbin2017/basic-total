# TODO LIST

![2021 BACKEND ROAD MAP](https://raw.githubusercontent.com/kamranahmedse/developer-roadmap/master/translations/korean/img/backend.png)

[이미지 출처](https://github.com/kamranahmedse/developer-roadmap/tree/master/translations/korean)

## 서적

- 🟢: 순항
- 🟡: 난항
- 🔴: 위기
- ⚫: 대기열

| 도서 명 | 구분 | 진도 | 목표 | 비고 | 시작 및 종료 기간 |
| --- | --- | --- | --- | --- | --- |
| 🟢 실전! 스프링5 를 활용한 리액티브 프로그래밍 | `JAVA`, `Reactive Programming` | 시작 | 리액티브 프로그래밍 이해 | 2021-08 ~ |
| 🟢 이펙티브자바 3판 | `JAVA` | 완독 | 완독 및 학습 내용 정리 | 주말 스터디 | 2021-01 ~ 2021-05 |
| 🟢 객체지향의 사실과 오해 | `OOP` | 4장 역할, 책임, 협력 | OOP 뽀개기 | - | 2021-03 ~ |
| 🟢 테스트 주도 개발 | `TDD`, `OOP` | 3부 테스트 주도 개발의 패턴 | TDD 학습 및 생활화 | - | 2021-03 ~ |
| 🟢 GoF 의 디자인 패턴 | 디자인 패턴 | 3부 생성 패턴 | [repository](https://github.com/psbin2017/like-multiplication-table/tree/master/src/main/java/com/multiplication/designpattern) | - | 2021-04 ~ |
| 🟡 자바 ORM 표준 JPA 프로그래밍 | JPA | 10장 객체지향 쿼리 언어 | 완독 및 도메인 설계와 친해지기 | 자꾸 회귀해서 인강 병행 | 2021-01 ~ |
| 🟡 스프링 부트와 AWS 로 혼자 구현하는 웹 서비스 | Totalize | 9장 Travis CI 자동 배포화 | 완독 및 실습 | Travis CI + Docker 연동하면서 삽질 중 (JPA 인강 때 적용하기) | 2021-01 ~ |
| 🔴 윤성우의 열혈 자료구조 | 자료구조 | 3장 링크드 리스트 | 완독 | 아는 내용 빠르게 넘어가고 진도 빼기 | 2021-03 ~ |
| ⚫ 스프링 마이크로 서비스 공작소 | Totalize | - | TODO | - | - |

## 인터넷 강의

| 강의 명 | 구분 | 진도 | 비고 | 시작 및 종료 기간 |
| --- | --- | --- | --- | --- |
| 🟡 Spring Cloud로 개발하는 마이크로서비스 애플리케이션 | Totalize | TODO | 2021-04 |
| 🟢 IntelliJ를 시작하시는 분들을 위한 IntelliJ 가이드 | IDE (Intelij) | done | **자주 복습 필요** | 2021-01 |
| 🟢 초보를 위한 도커 안내서 | 도커 | 쿡북 복습하기 | 2021-04 |
| 🟢 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발 | JPA | done | 실습 위주로 학습할 것 | 2021-03 |
| 🔴 성공적인 코딩 인터뷰를 위한 코딩 인터뷰 정복하기 - 코딩 테스트 | 자료구조와 알고리즘 | 섹션 4. 자료구조, 알고리즘 | 실습 위주로 학습할 것 | 2021-03 |

## 마주친 용어

- `READ` : 읽을 것
- `WRITE` : 정리할 것

| 단어 | 마주친 시기 | 수행 수준 |
| --- | --- | --- |
| JSR 133 (Java Memory Model) | 2021-04-26, 이펙티브 자바 11장 동시성 | `READ`, `WRITE` |

## 학습

- Java
  - [jdk 버전별 특징 ing](/java/version_feature.md)
  - GC 동작 방식과 특징
  - 멀티 스레드 프로그래밍
    - 락 분할(lock splitting)
    - 락 스트라이핑(lock striping)
    - 비차단 동시성 제어(nonblocking concurrency control)
    - 포크-조인 태스크 ... (병렬 스트림의 사용체)
  - JVM ++
  - OOP ++
  - 자료 구조 및 알고리즘
- Spring Boot (Spring)
  - Webflux
    - netty
  - Batch
  - Security (웹 보안 지식 항목)
  - Data Redis (캐시 항목)
  - RabbitMQ || Apache Kafka (메시지 브로커 항목)
  - WebSocket (웹 소켓 항목)
  - Test (테스트)
- Infra
  - AWS (클라우드 서비스)
    - EC2
    - S3
    - RDS ...
  - Docker
  - Kubernetes (인프라 오케스트레이션)
- Todo List
  - 표준 함수형 인터페이스 예제
  - 웹 크롤링
  - 리눅스 명령어
  - 온라인 세미나 듣기
  - Junit 에서 사용하는 어노테이션 (Web Application)

## 그 외

- GitBook 작성 내용 이관하기
- TODO 디렉토리 세분화? (복잡도 높아지면 고려하기)
- TODO markdown 템플릿화? (규칙없이 쓰고있어서 고민할 것)
