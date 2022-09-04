# 섹션 6: Docker Compose 우아한 다중 컨테이너 오케스트레이션

Docker Compose 가 무엇인지 왜 사용하는지 학습한다.

섹션 5 의 내용은 다양한 컨테이너를 만들기 위해 명령어를 정말 많이 사용하였다. 특히 개별 구성으로 각각 명령어를 실행하였다. Docker Compose 는 좀 더 편리하게 작동시키는 기능을 제공한다.

## Docker Compose

`docker run` 과 `docker build` 를 대체할 수 있다. 다중 컨테이너 애플리케이션을 시작하거나 중단할 수 있다. 커스텀 이미지를 만들기 위한 Dockfile 을 대체하지 않는다.

또한 한계점도 있는데 같은 **호스트에 있는 다중 컨테이너를 관리하는데 유용하나**, 다수 호스트로는 적합하지 않다. (개발자 입장에서는 로컬 환경까지는 유용하다는 거다.)

`docker-compose.yml` 또는 `docker-compose.yaml` 로 선언하여 사용한다.

```yml
## 도커 컴포즈의 사양 버전
version: "3.8"

## 컨테이너를 나열한다.
services:
    mongodb:
        ## 로컬 또는 도커 허브에서 조회된다.
        image: "mongo"
        volumes: 
            - data:/data/db
        environment:
            ## 둘다 지원한다.
            # MONGO_INITDB_ROOT_USERNAME: hello
            # - MONGO_INITDB_ROOT_USERNAME=hello
            - MONGO_INITDB_ROOT_USERNAME=hello
            - MONGO_INITDB_ROOT_PASSWORD=world
        ## env 파일을 추가하는 방법도 가능하다.
        # env_file:
            # - ./env/mongo.env

        ## 컨테이너 명을 지정할 수 있다.
        # container_name: mongodb

    backend:
        # backend 에 있는 Dockerfile 을 찾아 실행한다.
        build: ./backend
        # build:
            # context: ./backend
            # dockerfile: Dockerfile
            # args:
            #    some-arg: 1
        ports:
            -  "80:80"
        volumes:
            - logs:/app/logs
            ## 바인드 마운트는 경로가 간단해진다.
            - ./backend:app
            - /app/node_modules
        environment:
            - MONGO_INITDB_ROOT_USERNAME=hello
            - MONGO_INITDB_ROOT_PASSWORD=world

        ## 컴포즈만 있는 옵션으로 다른 컨테이너의 의존성을 명명한다.
        depends_on:
            - mongodb

    frontend:
        build: ./frontend
        ports:
            - "3000:3000"
        volumes:
            - ./frontend/src:/app/src
            - /app/nodes_modules
        ## -it 옵션
        stdin_open: true
        tty: true
        depends_on:
            - backend

## 명명된 볼륨은 이처럼 작성한다.
volumes:
    data:
    logs:
```

```env
MONGO_INITDB_ROOT_USERNAME=hello
MONGO_INITDB_ROOT_PASSWORD=world
```

실행은 `docker-compose up` 로 되고 네트워크과 볼륨은 명명하지 않았다면 프로젝트 디렉토리 명 _default 로 생성된다. `docker-compose down` 으로 종료시킬 수도 있다.

## 컨테이너 이름 이해하기

실제 실행된 컨테이너 명은 `${project-name}_${service}_1` 등으로 실행되는데 도커에 의해 서비스 명을 기억하고 생성된다. 따라서 컴포즈로 실행된 컨테이너는 이름이 다르니 주의할 것.

`container_name: mongodb` 등으로 추가 설정하면 실제 컨테이너 명도 지정할 수 있다.
