# Kafka

[출처 토크ON 77차](https://www.youtube.com/watch?v=VJKZvOASvUA&list=PL9mhQYIlKEheZvqoJj_PkYGA2hhBhgha8&index=2)

- 프로듀서/컨슈머 분리
- 메시지 데이터를 여러 컨슈머에게 허용
- 높은 처리량을 위한 메시지 최적화
- 스케일 아웃 가능
- 관련 생태계 제공

| 용어 | 설명 |
| --- | --- |
| Broker | 카프카 애플리케이션 서버 단위 |
| Topic | 데이터의 분리 단위, 하나의 토픽에는 N 개의 파티션을 가진다. |
| Partition | 레코드를 가지고 있다. 컨슈머가 요청하면 레코드를 전달한다. |
| Offset | 각 레코드 당 파티션에 할당된 고유 번호 |
| Consumer | 레코드를 polling 하는 애플리케이션 |
| Consumer Group | N 개의 컨슈머 묶음 |
| Consumer offset | 특정 컨슈머가 가져간 레코드 번호 |
| Producer | 레코드를 브로커로 전송하는 애플리케이션 |
| Replication | 파티션 복제 기능 |
| ISR | 리더+팔로워 파티션의 sync 묶음 |
| Rack-awareness | Server rack 이슈에 대응 |

## Kafka broker

- 실행된 카프카 애플리케이션 서버 중 1대
- 3대 이상의 브로커로 클러스터 구성
- 주키퍼와 연동(~2.5.0 version)
  - 주키퍼? 메타데이터(브로커 id, 컨트롤러 id 등) 저장
- n 개 브로커 중 1대는 컨트롤러(Controller) 기능 수행
  - 컨트롤러? 각 브로커에게 담당 파티션 할당 수행. 브로커 정상 동작 모니터링 관리. 누가 컨트롤러인지는 주키퍼에 저장.

## Record

객체를 프로듀서에서 컨슈머로 전달하기 위해 Kafka 내부 byte 형태로 저장 가능하도록 직렬화/역직렬화하여 사용.

- 기본 제공 직렬화 class: `StringSerializer`, `ShortSerializer`
- 커스텀 직별화 class 로 Custom Object 직렬화/역직렬화 가능

## Topic & Partition

- 메시지 분류 단위
- n개의 파티션 할당 가능
- 각 파티션마다 고유한 오프셋(offset)을 가짐.
- 메시지 처리 순서는 파티션 별로 유지 관리됨.

## Producer & Consumer

1. `프로듀서`는 레코드를 생성하고 `브로커`에 전송한다.
2. 전송된 레코드는 파티션에 신규 오프셋과 함꼐 기록된다.
3. `컨슈머`는 브로커로부터 레코드를 요청하여 가져감 (polling)

> 레코드를 생성하는 주체 `프로듀서`다.
>
> 레코드를 기록하는 주체는 `브로커`다.
>
> 레코드를 소비하는 주체는 `컨슈머`다.

## Kafka log and segment

> 실제로 메시지가 저장되는 단위는 **파일 시스템 단위이다.**

- 메시지가 저장될 때는 세그먼트 파일이 열려있다.
- 세그먼트는 시간 또는 크기 기준으로 닫힌다.
- 세그먼트가 닫힌 이후에 일정 시간(또는 용량)에 따라 삭제(delete) 또는 압축(compact)

## case

파티션 개수 >= 컨슈머 개수 ... 컨슈머가 더 많으면 일 안함

리밸런스: 파티션 컨슈머 할당 재조정

목적에 따라 컨슈머 그룹을 분리할 수 있다. 장애에 대응하기 위해서 재입수(재처리) 목적으로 임시 신규 컨슈머 그룹을 생성하여 사용하기도 한다.

애플리케이션 로그 적재용 컨슈머 그룹

## replication

Kafka broker 이슈에 대응하기 위해 사용방법은?  Partition 을 다른 Broker 에 복제하여 이슈에 대응한다.

> bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic topic_name --partition 3 --replication-factor 3

## 리더 파티션, 팔로워 파티션

- 리더 파티션: 클라이언트와 데이터를 주고 받는 역할
- 팔로워 파티션: 리더 파티션으로 부터 레코드를 지속 복제

특정 파티션의 리더, 팔로워가 레코드가 완벽하게 sync 가 맞는 상태 는 ISR(In-Sync Replica)

ISR 이 아닌 상태에서 장애가나면 `unclean.leader.election.enable`

Broker #1 장애 발생시?
    - partition #1 리더가 브로커 1 또는 브로커 2 중에 새로 할당 됨
    - 클라이언트는 새로운 파티션 리더와 연동

## Kafka Streams

- 데이터를 변환하기 위한 목적으로 사용하는 API
- 스트림 프로세싱을 지원하기 위한 다양한 기능을 제공한다.
  - Stateful 또는 Stateless 와 같이 상태 기반 스트림 처리 가능
  - Stream api 와 DSL(Domain Specific Language)를 동시 지원
  - Exactly-once 처리, 고 가용성 특징
  - Kafka security(acl, sasl 등) 완벽 지원
  - 스트림 처리를 위한 별도 클러스터 (ex, yarn 등) 불필요

## Kafka Connect

많은 경우 Kafka client 로 데이터를 넣는 코드를 작성하지만, Kafka Connect 로 data 를 import/export 할 수 있다.

- 코드 없이 configuration 으로 데이터를 이동시키는 목적
  - standalone mode, distribution mode
  - REST api interface 를 통해 제어
  - Stream 또는 Batch 형태로 데이터 전송가능
  - 커스텀 connector 를 통한 다양한 plugin 제공 (File, S3, Hive, Mysql etc)

## Kafka Mirror maker

특정 카프카 클러스터에서 다른 카프카 클러스터로 topic 및 record 를 복제하는 standalone tool

- 클러스터간 토픽에 대한 모든 것을 복제하는 것이 목적이다.
  - 신규 토픽, 파티션 감지 기능 및 토픽 설정 자동 sync 기능
  - 양방향 클러스터 토픽 복제
  - 미러링 모니터링을 위한 다양한 metric (latency, count 등) 제공
