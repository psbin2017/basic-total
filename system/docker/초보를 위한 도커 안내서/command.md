# 명령어

## 학습 내용 참고 출처

- [초보를 위한 도커 안내서](https://www.inflearn.com/course/%EB%8F%84%EC%BB%A4-%EC%9E%85%EB%AC%B8/dashboard)

## 학습 환경

- Windows pro 64bit

cmd 창

```text
docker version
```

## docker run

컨테이너를 실행한다.

> docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG..]

| `OPTIONS` | 설명 |
| --- | --- |
| `-d` | 백그라운드 모드 |
| `-p` | 호스트와 컨테이너의 포트를 연결 |
| `-v` | 호스트와 컨테이너의 디렉토리를 연결 |
| `-e` | 컨테이너 내 사용할 환경변수 설정 |
| `--name` | 컨테이너 이름 설정 |
| `-rm` | 프로세스가 종료되면 컨테이너 자동 제거 |
| `-it` | `-i` + `-t` 터미널 입력을 위한 옵션 |
| `--network` | 네트워크 연결 |

```text
// MYSQL
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \

// WORDPRESS
docker run -d -p 8080:80 \
  -e WORDPRESS_DB_HOST=host.docker.internal \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
```

## docker exec

실행된 컨테이너에 접속한다.

> docker exec [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG..]

```text
// MYSQL
docker exec -it mysql mysql
```

## docker ps

실행되고 있는 컨테이너 그리고 실행되었던 컨테이너 포함해 확인한다.

> docker ps, docker ps -a

## docker stop

실행 중인 컨테이너를 중지한다.

> docker stop [OPTIONS] CONTAINER [CONTAINER...]

## docker rm

종료된 컨테이너를 완전히 제거한다.

> docker rm [OPTIONS] CONTAINER [CONTAINER...]

## docker logs

컨테이너가 정상적으로 동작하는지 로그를 확인한다.

> docker logs [OPTIONS] CONTAINER

## docker images

도커가 다운로드한 이미지 목록을 본다.

> docker images [OPTIONS] [REPOSITORY[:TAG]]

## docker pull

이미지를 다운로드한다.

> docker pull [OPTIONS] NAME[:TAG|@DIGEST]

## docker rmi

이미지를 삭제한다. 단 컨테이너가 실행중인 이미지인 경우 삭제되지 않는다.

> docker rmi [OPTIONS] IMAGE [IMAGE...]

## docker network create

각 컨테이너끼리 이름을 통해서 통신할 수 있는 가상의 네트워크를 만든다.

> docker network create [OPTIONS] NETWORK

```text
docker network create app-network
```

## docker network option

컨테이너를 네트워크에 속하게 한다.

```text
// WORDPRESS
docker run -d -p 8080:80 \
  --network=app-network \
  -e WORDPRESS_DB_HOST=host.docker.internal \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
```

## volume mount (-v)

컨테이너를 삭제하면 안에 있던 데이터도 함께 삭제되는데 이를 해결하기 위한 방법으로 볼륨과 바인드 마운트가 있음.

-v 옵션을 통해 특정 볼륨으로 지정할 수 있다.

```text
docker stop mysql
docker rm mysql
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --network=app-network \
  --name mysql \
  -v /my/own/datadir:/var/lib/mysql \
```

// TODO 볼륨과 바인드 마운트

## docker-compose

앞서 살펴본 도커 명령어를 `.yml` 파일로 관리한다.

```text
version: '2'
services:
  db:
    image: mariadb:10.5
    volumes:
      - ./mysql:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
  wordpress:
    image: wordpress:latest
    volumes:
      - ./wp:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
```

| 명령 문 | 설명 |
| --- | --- |
| `image` | 이미지 명 |
| `volumes` | `-v` 볼륨 |
| `restart` | always ... 컨테이너가 죽으면 자동으로 실행 |
| `ports` | `-p` 포트 |
| `environment` | `-e` 환경 변수 |

## Dockerfile

파일 Dockerfile 을 통하여 일련의 도커 작업을 작성한다.

| 명령어 | 설명 |
| --- | --- |
| `FROM` | 기본 이미지 |
| `RUN` | 쉘 명령어 실행 |
| `CMD` | 컨테이너 기본 실행 명령어 (Entrypoint 인자로 사용) |
| `EXPOSE` | 오픈되는 포트 정보 |
| `ENV` | 환경변수 설정 |
| `ADD` | 파일 또는 디렉토리 추가. URL/ZIP 사용 가능 |
| `COPY` | 파일 또는 디렉토리 추가 |
| `ENTRYPOINT` | 컨테이너 기본 실행 명령어 |
| `VOLUME` | 외부 마운트 포인트 생성 |
| `USER` | `RUN` / `CMD` / `ENTRYPOINT` 를 실행하는 사용자 |
| `WORKDIR` | 작업 디렉토리 설정 |
| `ARGS` | 빌드타임 환경변수 설정 |
| `LABEL` | key - value 데이터 |
| `ONBUILD` | 다른 빌드의 베이스로 사용될 때 사용하는 명령어 |

## docker build

나만의 커스텀 도커 이미지를 만든다. [document](https://docs.docker.com/engine/reference/builder/)

> docker build -t {이미지명:이미지태그} {빌드 컨텍스트}

- `-f <Dockerfile 위치>` 로 옵션을 사용하여 다른 위치에 Dockerfile 파일 사용이 가능하다.
- `-t` 명령어로 도커 이미지 이름을 지정한다.
- `{네임스페이스}/{이미지이름}:{태그}` 형식

마지막에는 빌드 컨텍스트를 사용하는데 현재 디렉토리를 의미하는 점(.)을 주로 사용한다. 물론 다른 디렉토리도 지정 가능하다.

### case 1 install git

```text
// Dockerfile
FRON ubuntu:lastest
RUN apt-get update
RUN apt-get install -y git

// Command
docker build -t ubuntu:git-dockerfile .
docker images | grep ubuntu
```

### case 2 install node.js (webapplication)

```text
// Dockerfile
# 1. node 설치
FROM    ubuntu:20.04
RUN     apt-get update
RUN     DEBIAN_FRONTEND=noninteractive apt-get install -y nodejs npm

# 2. 소스 복사
COPY    . /usr/src/app

# 3. Nodejs 패키지 설치
WORKDIR /usr/src/app
RUN     npm install

# 4. WEB 서버 실행 (Listen 포트 정의)
EXPOSE 3000
CMD    node app.js
```

Dockerfile 은 이미지화 한 수준이나 명령어 순서에 따라 최적화를 할 수 있다. (시행착오가 있기 마련이다.)

## 이미지 푸쉬/풀

이미지 저장소(도커 허브)에 도커 이미지를 푸쉬하고 풀할 수 있다.

주의할 점은 무료 계정은 이미지가 private 하지 않아 모든 사용자가 사용할 수 있다.

```text
docker login
docker push {ID}/example
docker pull {ID}/example
```
