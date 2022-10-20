# DedupeResponseHeader

[document](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-deduperesponseheader-gatewayfilter-factory)

중복된 응답 헤더 키에 대한 핸들링. 예제처럼 ACA 관련 헤더들은 다른 서버를 경유할 때 애플리케이션 외부의 설정으로 붙는 경우가 있는데 브라우저 정책상 ACA 관련 헤더는 중복시 브라우저가 임의로 판단할 수 없어 예외가 발생한다. Strategy 를 설정하여 이를 해소가 가능하다.
