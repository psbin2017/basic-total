# 섹션 5: Docker로 다중 컨테이너 애플리케이션 구축하기

해당 모듈에서는 여러 빌딩 블록, 여러 서비스를 하나의 큰 앱과 결합하는 방법을 배운다.

- [DataBase] MongoDB: 데이터는 반드시 영구화, 엑세스 제한
- [Back-End] NodeJS REST API: 데이터는 반드시 영구화 (로그 파일), 라이브 리로드
- [Front-End] React SPA: 라이브 리로드

수행하기전에 `docker container prune` 으로 중지된 상태의 컨테이너를 미리 제거하자.

## v1 기본 백본

### MongoDB 도커화

```text
docker run mongo
    --name mongodb
    --rm
    -d
    -p 27017:27107
```

### Node 서버 도커화

```Dockerfile
FROM node:latest

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

CMD ["node", "app.js"]
```

```text
docker build
    -t goals-node .


docker run goals-node
    --name goals-backend
    --rm
    -d
    -p 80:80

// 인터널로 실행한 MongoDB 와 연결하려면,
// localhost:27017 이 아닌 host.docker.internal:27017 로 변경해야함
```

### React SPA 컨테이너로 옮기기

```Dockerfile
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "node", "start" ]
```

```text
docker build
    -t goals-react .

docker run goals-react
    --name goals-frontend
    --rm
    -d
    -p 3000:3000
    -it

// -it 를 선언하여 인터렉션 모드로 실행되게해야 한다.
```

## v2 네트워크 설정

```text
docker network create goals-net

// 몽고 디비
docker run mongo
    --name mongodb
    --rm
    -d
    --network goals-net

// 노드 서버
// 몽고 디비 컨테이너로 선언한 이름을 변경해야한다.
// host.docker.internal 을 mongodb 로 변경한다.
docker run goals-node
    --name goals-backend
    --rm
    -d
    -p 80:80
    --network goals-net

// 리액트 컨테이너
// 인터렉션을 위해서 해당 컨테이너는 포트를 노출한다.
docker run goals-react
    --name goals-frontend
    --rm
    -d
    --network goals-net
    -p 3000:3000
    -it
```

## v3 기능 개선

### MongoDB: 데이터 지속성, 보안 처리

몽고 디비의 데이터 지속성과 간단한 보안처리는 다음과 같다.

```text
// 몽고 디비
// 호스트 머신의 볼륨에 저장된다.
docker run mongo
    --name mongodb
    --rm
    -d
    --network goals-net
    -v data:/data/db
    -e 
        MONGO_INITDB_ROOT_USERNAME=hello
        MONGO_INITDB_ROOT_PASSWORD=world

// 노드 서버
// hello:world@mongodb:27017/course-goals?authSource=admin 로 이름과 비밀번호를 명명한다.
```

### Node 서버: 로그 데이터 지속성, 라이브 리로드

노드 서버의 로그 데이터 지속성 처리는 다음과 같다.

```Dockerfile
// nodemon 사용을 위해 실행 명령어를 변경한다.
// 추가로 앞선 작업에서 아이디 패스워드가 하드코딩되는 것을 개선하기 위해 변경
FROM node:latest

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=secret

CMD ["node", "start"]
```

```text
// 백틱으로 구문을 수정하고 하드 코딩된 값을 변경한다.
`mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb/`~~


// 그리고 .dockerignore 파일을 추가하여 이그노어 파일을 추가해준다.
node_modules
Dockerfile
.git


// 1. 라이브 리로드를 위한 바인드 마운트 (nodemon)
// 2. 로그를 위한 볼륨
// 3. 노드 모듈을 정상적으로 유지하기 위한 볼륨
docker run goals-node
    --name goals-backend
    --rm
    -d
    -p 80:80
    --network goals-net
    -v ${전체 경로를 기입}:/app
    -v logs:/app/logs
    -v /app/node_modules
    -e 
        MONGODB_USERNAME=hello
        MONGODB_PASSWORD=world
```

### React SPA: 라이브 리로드

```text
// .dockerignore 파일 추가
node_modules
Dockerfile
.git

// 1. 라이브 리로드를 위한 바인드 마운트
docker run goals-react
    --name goals-frontend
    --rm
    -d
    --network goals-net
    -p 3000:3000
    -it
    -v ${전체 경로를 기입}:/app/src
```

윈도우 환경에서 실행 안되면 wsl 문제이니 첨부된 자료 확인해서 반영할 것.
