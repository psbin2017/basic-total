# REST API

> REpresentational Safe Transfer

HTTP 통신에서 자원의 대한 요청 또는 처리에 네트워크 아키텍쳐이다.

## 구성

REST API 의 구성은 크게 3가지로 이루어진다.

- 자원 (Resource) - URI
- 행위 (Verb) - HTTP Method (GET/POST/PUT/PATCH/DELETE)
- 표현 (Representations) - Message (클라이언트와 서버와의 주고받은 데이터의 형태, json/xml 등)

## 특징

### 인터페이스 일관성 (Uniform Interface)

> 일관적인 인터페이스로 분리되어 있어야 한다.

클라이언트의 플랫폼에 종속적이지 않고 특정 언어나 기술에 국한되지 않는다. HTTP 플랫폼을 사용할 수 있는 모든 플랫폼에서 요청이 가능하다.

### 무상태성 (Stateless)

> 클라이언트의 컨텍스트가 서버에 저장되어서는 안된다.

이전 요청이 다음 요청에서 연관되어서는 안된다. 서버에 불필요한 정보를 관리하지 않는다.

### 캐시 가능 (Cacheable)

> 클라이언트는 응답에 대해 캐싱할 수 있어야 한다.

자세한 내용은 [다음](./cacheable.md)에서 다룬다.

### 계층 구조 (Layered System)

> 서버는 보안, 로드밸런싱, 암호화 등을 위한 계층을 추가하여 구조를 가질 수 있다.

### 서버 - 클라이언트 구조 (Server - Client)

> 서버와 클라이언트의 역할을 명확히 구분하며 독립성을 유지하고 의존성을 낮춘다.

### 자기 서술적 (Self Descriptiveness)

> REST 의 3가지 구성 요소로 응답을 사용하여 클라이언트가 메시지를 충분히 이해할 수 있어야 한다.

## 디자인

- URI 는 정보의 자원을 표현해야한다.
- 자원에 대한 행위는 HTTP Method 로 표현한다.

```text
-- 회원 삭제: 나쁜 케이스
GET /members/delete/1

-- 회원 삭제: 좋은 케이스
DELETE /members/1
```

URI는 자원을 표현하는데 중점을 두어야한다. 행위에 대한 표현이 들어가서는 안된다.

```text
-- 회원 조회: 나쁜 케이스
GET /members/show/1

-- 회원 조회: 좋은 케이스
GET /members/1
```

### 규칙

1. 슬래시 구분자(/)는 계층 관계를 나타내는데 사용한다.
   - URI 마지막 문자로 슬래시를 포함하지 않는다.
2. 하이픈 구분자(-)는 URI 가독성을 높이는데 사용한다.
   - 언더바 구준자(_)는 URI에 사용하지 않는다.
3. URI 경로는 소문자가 적합하다.
   - `RFC 3986 is the URI (Unified Resource Identifier) Syntax document` (URI 스키마와 호스트를 제외하고 대소문자를 구별하도록 규정하였다.)
4. 파일 확장자는 URI에 포함하지 않는다.
   - `Accept Header` 를 사용하여 포맷을 정한다.
5. 리소스간의 관계는 URI 로 표현할 수 있다.

#### 리소스간의 관계

```text
/리소스 명/리소스 ID/관계가 있는 다른 리소스명

GET /users/{userid}/devices
```

#### 자원을 표현하는 방법

자원 표현 방법은 크게 2가지로 나누어진다.

- Document : 문서 또는 객체
- Collection : 문서(객체)들의 집합

`http://restapi.example.com/sports/soccer` 는 sports 는 컬렉션으로 soccer 는 도큐먼트로 표현되었다고 생각하면 된다.

`http://restapi.example.com/sports/soccer/players/13` 는 sports 컬렉션, soccer 도큐먼트, players 컬렉션, 13 도큐먼트. 이 때 중요한 점은 컬렉션은 복수로 사용하고 있다. 직관적인 REST API 설계를 지향한다면 컬렉션과 도큐먼트를 사용할 때 단수 복수를 지켜주면 좋다.

## 학습 내용 참고 출처

- [위키 백과 REST](https://ko.wikipedia.org/wiki/REST)
- [Layered system constraint in REST API](https://stackoverflow.com/questions/30303116/layered-system-constraint-in-rest-api)
- [Caching REST API Response](https://restfulapi.net/caching/)
- [REST API 제대로 알고 사용하기](https://meetup.toast.com/posts/92)
