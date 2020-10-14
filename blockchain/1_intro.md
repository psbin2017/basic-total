# 개요

학습 내용 참고 출처

[HA와 OPS 그리고 RAC란?](https://codelib.tistory.com/23)

[하둡(Hadoop)의 HDFS에 대한 기본설명](https://yookeun.github.io/java/2015/05/24/hadoop-hdfs/)

| 용어 | 설명 |
| --- | --- |
| 블록 | 일정 기간에 발생한 거래 내역의 모음, 전체 거래 내역의 한 조각 |
| 블록체인 |블록을 시간에 따라 순서대로 체인 형태로 연결한 전체 거래 내역 |
| 분산 원장 | 거래 내역을 한 곳이 아닌 여러 곳에 분산해서 저장 |
| 탈중앙(DeCentralized)시스템 | 의사 결정이나 제어가 특정한 하나의 기관이나 서버가 아닌 다수의 합의에 의해 이루어지는 시스템 |
| P2P(Peer To Peer) 네트워크 | 중앙서버 없이 대등한 관계의 모든 노드와 노드 간을 연결한 네트워크 |
| 노드 | 서로 대등한 관계의 P2P 네트워크의 참여자 절반 이상의 합의가 필요하고, 함께 블록체인을 생성하고 배포하고 저장한다. |

## 네트워크 모델

Centralized : 단일 결정 센터
    - 부하 분산 (L4, 로드 밸런싱)
DeCentralized : 하나 이상의 결정 센터
Distributed : 결정 센터 없는 분산

## HA, OPS, RAC

`HA`(High Availability) 는 Active / Standby 로 서버를 구성하여 Active 서버 장애시 Standby 서버로 바꾸어 무중단 서비스를 구성할수 있도록한다.

`OPS`(Oracle Parallel Serve) 는 `HA` 의 문제점을 보완하였다.

- 한가로운 Standby 에 비해 바쁜 Active
- 특정 Active Instacne 가 장애시 다른 Active Instance 에게 위임
   1. CTF, TAF 등의 설정을 의미한다.
- Storage 를 추가하여 N 개의 Active Instance 가 공유하여 사용한다.
   1.`HA` 는 각 서버별로 각 Storage 를 가지고 있어 데이터 동기화에 문제가 있다.

`OPS` 의 Instance 1 에서 데이터를 변경하고 완료하였을 때 해당 내용을 Instance 2 에서 조회하려고 한다면 Instance 1 의 변경은 반드시 Storage 에 저장하여야한다. 이 과정은 디스크를 사용함에 시간이 오래 걸려 (Instance 1 에서 저장하기 이전까지 Instance 2 는 대기해야하는 지연) 성능이 저하된다.

`RAC`(Oracle Real Application Cluster)

Instance 1 과 Instance 2 를 연결하는 **Interconnect** 개념 도입

Storage 를 거치지 않고 메모리의 데이터를 **Interconnect** 를 통해 서로 즉시 교환한다. 이를 **Cash Fusion** 이라고도 한다.

> Oracle 8i = `OPS`,  Oracle 9i = `RAC`

여러 개의 Instance 는 마치 하나의 서버처럼 만들 수 있기 때문에 이를 **Cluster** 라고 한다.

- `GES`(Global Enqueue Service) : A Instance 에서 변경 작업을 하는 것과 B Instance 에서 변경 작업을 하는 것을 같이 관리하는 서비스
- `GCS`(Global Cache Fusion) : A Instance 에서 원하는 데이터를 요청하였을 때 찾지 못한 경우 B Instance 와의 **Interconnect** 를 통해 데이터를 전달할 수 있는 서비스
    1. Null(N) Mode: 해당 블록을 사용중인 사용자가 없다는 것을 의미
    2. Share(S) Mode: 해당 블록을 select 하고 있는 세션이 있다는 것을 의미
    3. Exclusive(X) Mode: 해당 블록을 update 하고 있다는 것을 의미

> Instance 간의 **Interconnect** 의 이동하는 빈도수와 전달하는 크기를 적절하게 튜닝하는 것이 성능 개선에 핵심이다.

## Hadoop Distributed File System

```text
[D1][D2][D3][D4][D5]
<B1>    <B1>    <B1>
    <B2><B2><B2>
        <B3><B3><B3>
// ...
```

`[D3]` 데이터 노드가 장애로 소실 되어도 `<B1>`, `<B2>`, `<B3>` 노트의 이용에는 지장이 없다.

- `Name Node` : 단일 결정 센터
- `Backup Node` : 2차 `Name Node`

## Inter Planetary File System

Peer To Peer 형태이다. 최초 서버가 데이터를 가지고 있지만 이후 공유를 하여 다른 클라이언트가 서버가 될 수 있다.

## Zookeeper

모든 노드는 동등해야한다. == 단일 결정 센터가 없어야 한다.

```text
[Follower][Follower][Leader][Follower][Follower]
    └─────────────────┘         │         │
            │         │         │         │
            └─────────┘         │         │
                      └─────────┘         │
                      └───────────────────┘
<Client><Client><Client><Client><Client><Client><Client>
```

Leader 의 기준은 없으며 Follower 의 누구든 Leader 가 될 수 있다.

Read 는 모두 동등하기에 특이사항이 없다. Update 는 변경을 공유해야하기 때문에 문제가 있다. 시간에 따라서 Leader 가 수행된 결과를 Follower 에게 공유하는 방법이다.

- 노드가 일정 이하 일정 이상이 되면 처리속도와 노드의 데이터 정합성이 깨지게 된다.
- 스토리지 기반이 아닌 메모리 기반이다.

## 복잡계

나무 구조 → 분기 과정 → 네트워크 (망) → 뿌리 줄기 구조 → 시간에 따른 구조 변화

이는 인간의 뇌구조와 유사하다.

### 복잡계 대칭

| 뉴턴식 사고관 | 복잡계적 사고관 |
| --- | --- |
| 환원주의: 작게 나누어 분석하게되면 전체를 이해할 수 있다. | 전일주의: 구성원의 상호작용을 포함하여 전체를 이해해야 한다. |
| 미래는 결정적이다. | 정해지지 않은 전개 가능성을 고려해야한다. |
| 복잡함은 복잡함을 낳을 뿐이다. | 복잡함이 간단함(새로운질서)를 낳을 수 있다. |
| 세상은 평형·선형적이다. | 세상은 비평형·비선형적이다. |
| 세상은 평형으로 돌아오는 안정적인 시스템이다. | 세상은 혼돈스럽고 불안정한 시스템이다. |
