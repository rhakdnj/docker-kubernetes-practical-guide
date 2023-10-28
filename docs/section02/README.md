# Docker Image & Container

## Dockerfile

도커에 의해 식별되는 특별한 파일이다.

'dockerfile'에는 자체 이미지를 빌드할 때 실행하려는 도커에 대한 명령이 포함된다.

`COPY` 명령어를 통해 로컬에 DockerFile 경로에서 상대 경로를 통해 특정 파일을 복사할 수 있다.

```dockerfile
# FROM baseImage: The name of the base image to use
FROM node

# .: 현재 경로 파일 및 디렉터리들 (dockerfile 제외)
# COPY [host file system] [image/container file system]
COPY . .
```

container가 될 때 자체 내부에 숨겨져 있는 파일 시스템이 존재한다.

이때 루트 폴더의 사용을 지양하고, 전적으로 사용자가 지정한 디렉터리를 사용하도록 하자.

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

이미지를 실행하는 것이 아닌, 이미지를 기반으로 컨테이너를 실행하는 것이다.

**CMD: Provide defaults for an executing container. If an executable is not specified, then `ENTRYPOINT` must be
specified as well. There can only be one `CMD` instruction in a `Dockerfile`**

이미지가 생성될 때 실행되지 않고, 이미지를 기반으로 컨테이너가 시작될 때 실행된다.

다만 CMD는 배열로 전달해야 한다.

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

- EXPOSE 포트를 명시했다고 달라지는 건 없다.
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

<br/>

## 이미지 레이어 이해하기

이미지를 빌드할 때, 변경된 부분의 명령과 그 이후의 모든 명령이 재평가된다.

```dockerfile
FROM node
WORKDIR /app
COPY . /app
RUN npm install
CMD ["node", "server.js"]
```

만약 소스 코드의 변경이 발생한다면 `COPY . /app`지점부터 변경 점이라고 인식한다.

이에 따라 그 이후의 모든 명령어는 캐시를 사용하지 않고 다시 빌드된다.

이를 레이어 기반 아키텍처라고 한다.

모든 명령은 Dockerfile의 레이어를 나타낸다.

이미지 레이어를 기반으로 컨테이너를 생성하고 마지막의 `CMD` 명령어를 통해 컨테이너를 실행한다.

이를 하나의 레이어가 변경될 때마다 모든 후속 레이어도 다시 실행되므로 이를 피해야한다.

```dockerfile
FROM node
WORKDIR /app
COPY package.json /app
RUN npm install
COPY . /app
CMD ["node", "server.js"]
```

위와 같이 작성하면 `COPY . /app` layer까지 docker 생성한 캐시를 재사용 할 수 있다.

<br/>

## 컨테이너 중지 & 재시작

- `docker ps`: 모든 컨테이너를 리스트
- `docker ps -a`: 더 이상 실행되지 않는 중지된 컨테이너를 포함 리스트
- `docker start <container(Id or Name)>`: 중지된 컨테이너를 재시작
    - `docker run`: 이미지를 기반으로 새로운 컨테이너를 생성, 재시작의 느낌이 아니다.

<br/>

## Attached & Detached 컨테이너

`docker run -p 3000:80 <containerId>`를 사용하면 터미널에서 프로세스를 만들고 막히는 것을 확인할 수 있다.

이 프로세스는 이 터미널을 차단하고 있지만, 즉 foreground에서 실행되고 있는 것이다.

이는 사용자가 직접 제어할 수 있다.

`docker start`는 default 모드가 -d(detached) 이다.

`docker run`으로 실행하는 경우, attached 모드가 default이다.

`docker run -d`로 실행하면 detached로 실행할 수 있다.

이후 attach 모드로 돌아가려면, `docker attach <container(id or name)>`

혹은 `docker logs -f <container(id or name)>` 로그를 follow-up 할 수 있다.

<br/>

## 인터렉티브 모드로 들어가기

`-i, --interactive`: Keep STDIN open even if not attached

`-t, --tty`: Allocate a pseudo(의사)-TTY

- `docker run -it <containerId / containerName>`
- `docker start -a -i <containerId / containerName>`

<br/>

## 이미지 & 컨테이너 삭제하기

`docker ps -a` 모든 이력의 컨테이너가 확인된다.

때때로 이 목록을 정리하고, 더 이상 필요하지 않은 컨테이너를 제거하는 것이 합리적이다.

이때, `docker rm <containerId, containerName>` 다만 `docker stop`이 선행되어야 한다.

`docker images`: 이미지를 리스팅

`docker rmi <iamgeId>`: 이미지를 삭제

`docker image prune <imageId>`: 다만, 이미지를 삭제할 때 이미지를 통해 만든 컨테이너를 삭제해야한다. 이 때 사용되는 명령어이다.

<br/>

## 중지된 컨테이너 자동 제거하기

`--rm`: Automatically remove the container when it exits

`docker run -p 3000:80 -d --rm <containerId / containerName>`

일반적으로 다시 시작학지 않으모르 `--rm`을 추가하자.

<br/>

## 이미지 살펴보기

`docker image inspect <imageId>`

- 생성 날짜
- Env
- CMD = EntryPoint
- ...
- Docker version
- Image Layer

<br/>

## 컨테이너에 혹은 컨테이너로부터 파일 복사하기

실행중인 컨테이너에 무언가를 추가하거나, 무언가를 추출하고 싶다면 어떻게 해야 할까요?

`cp`: 실행중인 컨테이너로 또는 실행중인 컨테이너 밖으로 파일 또는 폴더를 복사할 수 있다.

`docker cp dummy/. <containerId, containerName>:/<targetDir>`
`docker cp <containerId, containerName>:/<targetDir> dummy`

<br/>

## 컨테이너와 이미지에 이름 지정 & 태그 지정하기

`docker run -p 3000:80 -d --rm --name goalsapp <imaegId, imageName>`

이미지 또한 이름(tag)를 지정할 수 있다.

크게 `name:tag`로 이뤄진다. 

- name: 전문화된 이미지 그룹을 정의한다. i.g.) "node"
- tag: 이미지 그룹 내 특수 이미지를 정의한다. i.g.) "18"

`docker build -t goals:latest .`

if remove all images, `docker image prune -a` 

<br/>

## 이미지 공유하기

이미지를 푸시(push)할 수 있는 두 가지 주요 위치가 있다. 도커 허브 또는 Private Registry이다.

- Share: `docker push <imageName>`
- Use: `docker pull <imageName>`

- 생성 시점: `docker build -t chungjm0711/node-hello-world .`
- 기존에 있을 때: `docker tag node-demo:latest chungjm0711/node-hello-world:latest`

<br/>

## 모듈 요약

Docker는 이미지와 컨테이너에 관한 모든 것이다.

이미지는 컨테이너의 템플릿 청사진이며, 이미지를 기반으로 여러 컨테이너를 생성할 수 있다.

이미지는 다운로드(docker pull)되거나 `Dockerfile` 및 `docker build`를 사용하여 생성된다.

이미지는  빌드 속도(캐싱!) 및 재사용성을 최적화하기 위해 여러 레이어(명령 1개 = 레이어 1개)로 구성 된다.

컨테이너는 docker `run Image`로 생성되며 다양한 옵션 플래그로 구성할 수 있다.

컨테이너를 나열(`docker ps`)하고 제거(`docker rm`)하고 중지 + 시작(`docker stop start`)할 수 있다.

이미지는 나열(`docker Images`), 제거(`docker rmi`, `docker image prune`) 및 공유(`docker push pull`)될 수도 있다.
