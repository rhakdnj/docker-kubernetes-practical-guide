# Docker Image & Container

## Dockerfile

도커에 의해 식별되는 특별한 파일이다.

'dockerfile'에는 자체 이미지를 빌드할 때 실행하려는 도커에 대한 명령이 포함된다.

`COPY` 명령어를 통해 로컬에 DockerFile 경로에서 상대 경로릍 통해 특정 파일을 복사할 수 있다.

```dockerfile
# FROM baseImage: The name of the base image to use
FROM node

# .: 현재 경로 파일 및 디렉터리들 (dockerfile 제외)
# COPY [host file system] [image/container file system]
COPY . .
```

container가 될 때 자체 내부에 숨겨져 있는 파일 시스템이 존재한다.

이 때 루트 폴더의 사용을 지양하고, 전적으로 사용자가 지정한 디렉터리를 사용하도록 하자.

`/app` 폴더가 존재하지 않는다면, 이미지와 컨테이너에서 생성한다.

```dockerfile
FROM node
COPY . /app
```

**RUN: Execute commands inside a shell**

```dockerfile
FROM node
COPY . /app
RUN npm install
```

근데 위와 같이 `RUN` 명령어를 실행하는 위치가 컨테이너 파일 시스템의 루트 폴더이다.

이에 따라 작업 폴더를 명시해야 한다.

**WORKDIR: The absolute or relative path to use as the working directory. Will be created if it does not exist**

```dockerfile
FROM node
WORKDIR /app
# /app -> . (현재 작업 디렉터리를 의미)
COPY . .
RUN npm install
```

server를 이제 띄울 때 RUN 명령어를 사용한다면 이는 잘못된 방법이다.

이는 RUN 명령은 빌드될 때마다 실행되기 때문이다.

즉, dockerfile에서 명령어는 이미지 설정에 관한 것이다. (이미지는 컨테이너의 템플릿이다.)

이미지를 실행하는 것이 아닌, 이미지를 기반으로 컨테이너를 실행하는 것 이다.

**CMD: Provide defaults for an executing container. If an executable is not specified, then `ENTRYPOINT` must be
specified as well. There can only be one `CMD` instruction in a `Dockerfile`**

이미지가 생성될 때 실행되지 않고, 이미지를 기반으로 컨테이너가 시작될 때 실행된다.

다만 CMD는 배열로 전달해야한다.

```dockerfile
FROM node
WORKDIR /app
# /app -> . (현재 작업 디렉터리를 의미)
COPY . .
RUN npm install
#RUN node server.js
CMD["node", "server.js"]
```

**EXPOSE: The port that this container should listen on at runtime.**

- EXPOSE 포트를 명시했다고 달라지는건 없다.
- 'docker run -p' 옵션으로 포트를 명시하지 않으면 컨테이너의 포트가 호스트 운영체제에 공개되지 않는다.
- EXPOSE 구문으로 명시한 포트는 'docker run -P' 명령을 이용할 때 호스트 운영체제로 오픈됩니다.
- 이 때, 호스트 운영체제의 랜덤 포트 번호가 컨테이너의 EXPOSE 구문으로 명시한 포트에 매핑됩니다.

```dockerfile
FROM node
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 80
CMD ["node", "server.js"]
```

자체 이미지를 만들어본다.

`build` 명령어는 `dockerfile`을 통해 새 커스텀 이미지를 빌드하도록 도커에게 지시한다.

```shell
# 상대 경로에 dockerfile
$ docker build .
5b803c7b513d61

docker run -p 3000:80 5b803c7b513d61
```

-p, --publish-all: 노출된 모든 포트를 임의의 포트에 게시
