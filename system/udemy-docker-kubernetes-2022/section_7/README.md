# 섹션 7: "유틸리티 컨테이너"로 작업하기 & 컨테이너에서 명령 실행하기

공식적인 용어는 아니나 애플리케이션 코드 없이 만드는 컨테이너를 지칭한다.

- 애플리케이션 컨테이너 : 환경 / 애플리케이션 코드 (docker run mynpm)
- 유틸리티 컨테이너 : 특정 환경에 종속 (docker run mynpm init)

## 오버라이드

`docker run node -it` 를 실행시 인터랙티브 모드로 노드 이미지를 가져온다. 이 경우는 이미 지정된 이미지의 명령어가 자체적으로 실행 될 것이다.

`docker run node -it npm init` 을 실행한다면, 지정된 이미지가 아닌 `npm init` 명령으로 프로젝트를 생성하고 사용자의 오버라이드가 가능하다. (물론 활용은 하단 섹션으로 진행한다.)

## 유틸리티 컨테이너 (명령어를 통한 방법)

```Dockerfile
FROM node:14-alpine

WORKDIR /app

// 엔트리포인트는 항상 컨테이너 이름 뒤에 붙어서 실행된다.
// CMD 는 실행된 뒤에 실행되는 것과 다르다.
ENTRYPOINT [ "npm" ]
```

```text
docker run
    -it
    -v ${절대경로}:app 
    node-util
    npm init
```

## 유틸리티 컨테이너 (컴포즈를 통한 방법)

```Dockerfile
FROM node:14-alpine

WORKDIR /app

ENTRYPOINT [ "npm" ]
```

```yml
version: "3.8"

services:
    npm:
        build: ./
        stdin_open: true
        tty: true
        volumes:
            - ./:/app
```

```text
docker-compose up init

docker-compose exec

// run 은 특정 서비스를 단일로 실행할 수 있다.
docker-compose run --rm npm init
```

## 컨테이너/이미지 명령어

### `docker exec`

> 실행 중인 내의 컨테이너에서 특정 명령어를 실행 할 수 있다.

node 앱이라는 가정 `docker exec ${containerName} npm init` npm 명령어를 실행 할 수 있다.
