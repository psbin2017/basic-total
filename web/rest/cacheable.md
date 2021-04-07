# cacheable

`GET` 요청은 캐싱이 가능하다. `POST` 는 기본적으로는 캐시할 수 없지만, 명시적으로는 캐시를 허용하는 지시문이 있다. `PUT` 과 `DELETE` 는 사용할 수 없다.

캐싱 동작을 제어하는 데 사용할 수 있는 HTTP 응답 헤더는 다음과 같다.

## Expires

> `Expires: <http-date>`
>
> `Expires: Fri, 20 May 2016 19:20:49 IST`

"max-age" 혹은 "s-max-age" 디렉티브를 가진 `Cache-Control` 있으면 무시된다.

## Cache-Control

`Cache-Control` 은 요청 디렉티브와 응답 디렉티브를 달리 가진다.

> `Cache-Control: max-age=<seconds>`
>
> `Cache-Control: max-age=3600`

자세한 내용은 [MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Cache-Control) 을 참조하자.

캐싱을 막는 방법은 `Cache-Control: no-cache, no-store, must-revalidate` 이 있다.

## ETag

> `ETag: "<etag_value>"`
>
> `ETag: "abcd1234567n34jv"`

## Last-Modified

> `Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT`
>
> `Last-Modified: Fri, 10 May 2016 09:17:49 IST`

## 출처

- [MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers)
- [CACHING](https://restfulapi.net/caching/)
