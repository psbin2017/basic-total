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
