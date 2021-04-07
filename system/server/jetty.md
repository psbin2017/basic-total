# 0. 개요

자바 웹 애플리케이션 서버를 선택할 때 Tomcat, Jetty, JBoss, GlassFish, WildFly, TomEE, WebLogic, WebSphere 등과 같은 선택을 할 수 있다. 이들의 간략한 설명을 하고 선택 방법과 Jetty 설정에 대해 다루어 본다.

## 1. Tomcat

- Apache 에서 가장 널리 사용되는 오픈 소스 프로젝트이다.
- 강력한 개발자 커뮤니티(stack overflow)에 설명이 잘 되있다.
- 수년 동안 다양한 버전에 따라 테스트가 되었고 입증 되었으며 안정적이다.
- 많은 기업 및 정부 기관이 사용함에 따라 상업적 성공을 거두었다.
- 스프링과 같은 다른 웹 애플리케이션 프레임워크와의 통합도 가볍다.
- 서블릿 컨테이너를 찾고있다면 Tomcat 은 확실한 강자이다.
- TomEE, JBoss, WildFly 를 통한 엔터프라이즈급 지원으로 확장이 가능하다.

## 2. Jetty

- 메모리를 적게 사용하며 가볍고 속도와 확장성을 제공한다.
- 자바 애플리케이션뿐만 아니라  전화기 및 셋업 박스에서 쉽게 임베드되고 비동기 서버로 사용할 수 있다.
- Eclipse 재단의 좋은 커뮤니티를 가진 오픈 소스이다.
- 작은 설치 공간을 가져 신속하게 다시 시작할 수 있다.
- 높은 수준의 커스터마이징 기능을 제공하느 플러그인 형
- GWT, JRuby, Grils, Scala / Lift 등과 같은 여러 프레임 워크에 내장

## 3. GlassFish

- 오라클이 개발한 JavaEE 애플리케이션 서버이다.
- Tomcat 과 Jetty 에 비해 다소 무겁다.
- 무거운 만큼 조작하기 까다롭다.
- 항상 최신의 Java EE 기능을 지원한다.
- 무료 상용 지원이 부족하여 금액을 지불해야 한다.

## 4. WildFly

- JBoss 애플리케이션 서버로 알려져 있었고 Red Hat 에서 개발한 애플리케이션 서버이다.
- 상업적 JBoss 엔터프라이즈 애플리케이션 플랫폼에 서버 마이그레이션이 쉽다.
- 필요에 따라 JBoss EAP 로 신속한 마이그레이션이 가능하다.

## 5. 선택 방법

선택의 가장 큰 요소는 JavaEE 의 지원에 대한 깊이이다. JavaEE 지원이 필요하지 않다면 Tomcat 이나 좀더 가볍고 제한된 환경에서는 Jetty 로 충분하다.

그렇다고 JavaEE 에 모든 기능을 사용하지 못하는 것은 아니다. 일부 의존성을 포함시키고 사용한다면 Tomecat 이나 Jetty 또한 좋은 선택이 될 수 있다. 그러나 의존성이 대다수의 JavaEE 지원이 필요할 정도로 요구된다면, WildFly 를 권고한다.

마지막으로 가장 중요한 점은 조직에서 이미 정해진 애플리케이션이 있다면 따르는게 바람직할 것이다.

## 6. Jetty 설정하기

### 1. 공통 주의 사항

공통 Filter 의존성을 가지고 있는 프로젝트의 경우 (implements Filter) web.xml 에서 해당 필터기능을 주석처리해야한다. 제티는 빌드 된 target 폴더를 바라보기 때문에 auto build를 켜두어야 한다. 가급적 config.properties 가 변경되는 경우에는 서버를 다시 실행 하는 것을 권고한다.

### 2. 제티 종류

이클립스 IDE 기준

#### 1. 노랑 제티(임베디드 제티)

마켓 플레이스에서 노랑 제티 설치후 Run Configuration 에서 설정해서 실행한다. Show Debug에서 Terminate and Relaunch 등등으로 서버 제어 가능

#### 2. 파랑 제티(웹 애플리케이션 제티)

마켓 플레이스에서 파랑 제티 설치후 Preferences > Server > 에서 설치 버전에 맞는 파랑 제티 추가

#### 3. 메이븐 제티(메이븐 임베디드 제티)

메이븐 의존성으로 설치하여 처리한다. 안해봤다. (의존성이 붙고 만약 기존 서버 연동이 있다면 비추천)

### 3. 설정

**jetty plugin download**.

```text
mvn dependency:get \
-DgroupId=org.eclipse.jetty \
-DartifactId=jetty-maven-plugin \
-Dversion=9.1.0.RC2
```

**실행**.

```text
mvn -Djetty.reload=automatic \
-Djetty.scanIntervalSeconds=1000 \
org.eclipse.jetty:jetty-maven-plugin:run
```

## 7. 더 봐야할 것들

### 1. Undertow

Spring Boot 공식 지원 내장인 WAS 이다. Tomcat 코드를 이용해 동일한 성능을 낼 수 있다.

## 8. 학습 내용 참고 출처

- TODO ... 어디였는지 적지 않아서 알 수 없음
