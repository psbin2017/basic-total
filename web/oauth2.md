# OAuth 2.0

> 외부 서비스에서도 인증을 가능하게 하고 그 서비스의 API를 이용하게 해주는 것, 이것을 바로 OAuth 이다.

인증과 인가를 획득하는 것으로 볼 수 있다.

- 인증 (Authentication): 사용자가 누구인지 확인하는 절차
- 인가 (Authorization): 사용자를 식별하여 알맞은 권한을 부여

## OAuth 1.0 diff OAuth 2.0

1. OAuth 2.0 은 기능이 단순해졌으며 인증 서버의 분리와 확장이 지원되도록 바뀌었다.
   - OAuth 2.0 은 웹 애플리케이션만 국한되지 않고 애플리케이션으로 지원이 강화되었다.
   - OAuth 2.0 은 JWT Bearer 토큰 인증방식을 사용하여 signature 가 필요없어져 간단해졌다
2. OAuth 2.0 은 HTTPS(SSL) 이 기본이다.
3. OAuth 2.0 은 다양한 방식의 인증 방식을 제공한다. (OAuth 1.0 은 HMAC 암호화)
4. OAuth 2.0 은 Access Token 이 Lift-time 을 가져 만료가 가능해졌다.

## 구성

1. 사용자/자원 소유자 (Resource Owner): 사용자
2. 애플리케이션 (Client): 호스팅 서버에서 제공하는 자원을 사용하는 애플리케이션
3. 호스팅 서버 (Resource Server/API Server): 자원을 호스팅하는 서버
4. 인증 서버 (Authorization Server): 사용자 동의를 받아 권한을 부여하는 서버

## 프로세스

![Oauth 2.0 프로세스](images/ouath2_image01.jpg)

이미지 출처: [PAYCO 개발가이드](https://developers.payco.com/guide/development/start)

1. `사용자`가 `서비스`를 접근하여 로그인을 요청한다.
2. `서비스`가 `인증 서비스`에 로그인을 요청한다.
3. `인증 서비스`는 `사용자`에게 로그인 페이지를 제공한다.
4. `사용자`는 로그인 페이지로 로그인한다.
5. `인증 서비스`는 Authorization Code 를 `사용자`에게 반환한다.
6. `사용자`는 Authorization Code 를 가지고 `서비스` 의 Redirect Page 로 이동한다.
7. `서비스`는 Authorization Code 로 `인증 서비스`에게 Access Token 을 요청한다.
8. `인증 서비스`는 Access Token 을 발급하여 `서비스`에게 전달한다.
9. `서비스`는 Access Token 을 저장한다.
10. `서비스`는 `사용자`를 인증한다.

## 인증 종류

OAuth 2.0 인증 종류는 크게 4 가지가 있다.

### Authorization Code Grant

- 서버사이드 코드로 인증하는 방식
- 권한서버가 클라이언트와 리소스서버간의 중재역할.
- Access Token 을 바로 클라이언트로 전달하지 않아 잠재적 유출을 방지.
- 로그인시에 페이지 URL에 response_type=code 라고 넘긴다.

### Implicit Grant

- token 과 scope 에 대한 스펙 등은 다르지만 OAuth 1.0a 과 가장 비슷한 인증방식
- Public Client 인 브라우저 기반의 어플리케이션(Javascript application)이나 모바일 어플리케이션에서 이 방식을 사용하면 된다.
- OAuth 2.0 에서 가장 많이 사용되는 방식이다.
- 권한코드 없이 바로 발급되서 보안에 취약
- 주로 Read only 인 서비스에 사용.
- 로그인시에 페이지 URL에 response_type=token 라고 넘긴다.

### Password Credentials Grant

- Client 에 아이디/패스워드를 저장해놓고 아이디/패스워드로 직접 Access Token 을 받아오는 방식이다.
- Client 를 믿을 수 없을 때에는 사용하기에 위험하기 때문에 API 서비스의 공식 어플리케이션이나 믿을 수 있는 Client 에 한해서만 사용하는 것을 추천한다.
- 로그인시에 API 에 POST 로 grant_type=password 라고 넘긴다.

### Client Credentials Grant

- 어플리케이션이 Confidential Client 일 때 id 와 secret 을 가지고 인증하는 방식이다.
- 로그인시에 API에 POST로 grant_type=client_credentials 라고 넘긴다.

## Token

### Access Token

인증 종류에 상관없이 모두 정상적인 경우 클라이언트에게 Access Token 이 발급된다. 이 값은 보호된 리소스 접근시 권한 확인용으로 사용된다. ID 와 Password 를 이 토큰으로 표현하며 리소스 서버에서는 여러 인증 방식을 대응하지 않아도 권한을 확인할 수 있게 된다.

### Refresh Token

발급된 Access Token 은 Life-time 을 가지고 있다. 만료될 시 정상 결과를 반환하던 인증이 실패한다. Access Token 발급시 같이 받은 Refresh Toekn 으로 인증 서버에 전달하여 새로운 Access Token 을 발급 받는다. 새로운 Access Token 발급시 Refresh Token 도 새로 발급 될 수 있다.

## 학습 내용 참고 출처

- [OAuth 1.0 과 OAuth 2.0 차이점](https://berrrrr.github.io/programming/2019/11/03/oauth1-vs-oauth2/)
- [OAuth 란 무엇일까](https://showerbugs.github.io/2017-11-16/OAuth-%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)
