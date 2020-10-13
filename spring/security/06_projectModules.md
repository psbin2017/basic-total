# Project Modules

원글 출처 : [Spring security](https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/)

번역 출처 : [토리맘의 한글라이즈 프로젝트](https://godekdls.github.io/Spring%20Security/contents/)

## Core

`spring-security-core.jar`

핵심 인증 기능과 접근 제어를 담당하는 클래스와 인터페이스가 있으며, 원격 지원과 기본적인 프로비저닝 API 를 제공한다.

## Remoting

`spring-security-remoting.jar`

Spring Remoting 과 통합을 지원한다.

## Web

`spring-security-web.jar`

웹 보안 기반 코드를 제공한다.

서블릿 API 의존성이 있는 코드는 여기에 있으며 웹 인증 서비스나 URL 기반 접근 제어가 필요하다면 해당 모듈을 사용하면 된다.

## Config

`spring-security-config.jar`

보안 네임스페이스를 파싱하는 코드와 자바 설정 코드를 제공한다.

config 패키지 하위 클래스는 직접 사용하는 용도가 아니다

## LDAP

`spring-security-ldap.jar`

LDAP 인증과 프로비저닝 코드를 제공한다.

## OAuth 2.0 Core

`spring-security-oauth2-core.jar`

OAuth 2.0 프레임워크와 OpenID Connect Core 1.0 지원을 위한 핵심 클래스와 인터페이스가 있다.

클라이언트든, 리소스 서버든, 인가 서버든, OAuth 2.0이나 OpenID Connect Core 1.0을 사용한다면 이 모듈이 필요하다.

## OAuth 2.0 Client

`spring-security-oauth2-client.jar`

OAuth 2.0 프레임워크와 OpenID Connect Core 1.0 클라이언트를 지원한다.

OAuth 2.0 로그인이나 OAuth 클라이언트를 사용하는 어플리케이션에 필요하다.

## OAuth 2.0 JOSE

`spring-security-oauth2-jose.jar`

JOSE (Javascript Object Signing and Encryption) 프레임워크를 지원한다.

JOSE 프레임워크는 양단 간 클레임을 안전하게 전송할 수 있는 방법을 제공하는 프레임워크다.

다음과 같은 스펙으로 구성되어 있다.

- JSON Web Token (JWT)
- JSON Web Signature (JWS)
- JSON Web Encryption (JWE)
- JSON Web Key (JWK)

## OAuth 2.0 Rersource Server

`spring-security-oauth2-resource-server.jar`

OAuth 2.0 리소스 서버를 지원한다.

OAuth 2.0 Bearer 토큰으로 API를 보호할 때 사용한다.

## ACl

`spring-security-acl.jar`

도메인 객체 ACL 구현에 특화된 구현체를 제공한다.

애플리케이션에 있는 특정 도메인 객체 인스턴스를 보호할 때 사용한다.

## CAS

`spring-security-cas.jar`

시큐리티의 CAS 클라이언트 통합을 지원한다.

CAS 통합 인증 (single sign-on) 서버에서 스프링 시큐리티의 웹 인증을 사용할 때 필요하다.

## OpenID

`spring-security-openid.jar`

OpenID 웹 인증을 지원한다.

외부 OpenID 서버에서 사용자를 인증할 때 사용한다.

## Test

`spring-security-test.jar`
