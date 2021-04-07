# HTTP 메소드

HTTP 메소드로는 `GET` `POST` `PUT` `PATCH` `DELETE` 가 있다. CRUD 에 매핑되는 메소드는 다음과 같다.

| CRUD | 메소드 |
| --- | --- |
| C | `POST` |
| R | `GET` |
| U | `PUT`, `PATCH` |
| D | `DELETE` |

## 멱등성

> 멱등성(冪等性, 영어: idempotent)은 수학이나 전산학에서 연산의 한 성질을 나타내는 것으로, 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미한다.

`POST` 를 제외한 나머지 메소드는 멱등성을 가져야 한다.

## POST diff PUT

POST 는 요청마다 새로운 리소스가 생성된다.

```text
POST /member
{
    "name" : "라이언",
    "type" : "KAKAO"
}

200 OK
{ 
    "id" : 1,
    "name" : "라이언",
    "type" : "KAKAO"
}

POST /member
{
    "name" : "라이언",
    "type" : "KAKAO"
}

200 OK
{ 
    "id" : 2,
    "name" : "라이언",
    "type" : "KAKAO"
}
```

PUT 은 요청마다 같은 리소스를 반환한다. 요청마다 리소스 안의 속성은 변경된다.

```text
PUT /member/1
{
    "name" : "라이언",
    "type" : "KAKAO"
}

200 OK
{ 
    "id" : 1,
    "name" : "라이언",
    "type" : "KAKAO"
}

PUT /member/1
{
    "name" : "무지",
    "type" : "KAKAO"
}

200 OK
{ 
    "id" : 1,
    "name" : "무지",
    "type" : "KAKAO"
}
```

## PUT diff PATCH

U 에 대한 대응 메소드가 2개 인데, `PUT` 과 `PATCH` 는 다음의 차이가 있다.

```text
PUT /member/1
{
    "name" : "제이지"
}

200 OK
{
    "name" : "제이지",
    "type" : null
}
```

`PUT` 보내지 않은 속성은 null 이 된다.

```text
PATCH /member/1
{
    "name" : "제이지"
}

200 OK
{
    "name" : "제이지",
    "type" : "KAKAO"
}
```

`PATCH` 는 보낸 속성 값만 수정된다.

따라서, 전체 속성에 대한 수정이 필요한 경우는 `PUT` 아닌 일부 속성에 대한 수정은 `PATCH` 가 바람직하다.
