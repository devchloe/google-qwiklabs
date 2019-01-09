# Overview


> With Docker, you can separate your applications from your infrastructure and treat your infrastructure like a managed application.


Docker는 인프라스트럭처를 하나의 어플리케이션으로서 다루며 관리할 수 있다? 우분투와 같은 OS를 도커 이미지로 만들고 실행할 수 있어서 그런가?

> Docker does this by combining kernel containerization features with workflows and tooling that helps you manage and deploy your applications.

정확하게 kernel containerization이란 무엇일까? 호스트 OS의 자원, 프로세스, 사용자 등과 같은 것을 격리시키는 기술을 이용해서 호스트 리소스를 그대로 사용하지만 격리하는 것을 의미하나?

> Docker containers can be directly used in Kubernetes, which allows them to be run in the Kubernetes Engine with ease. 

Docker container = containerized application
이 것을 Kubernetes에 올려서 운영하는 것까지 한 사
이클로 보는 것
이겠지


## Setup

```bash
$ gcloud auth list # active account 확인 
$ gcloud config list project # project ID 확인 
# more at https://cloud.google.com/sdk/gcloud
```

## Hello World

### docker run
>  The docker daemon searched for the hello-world image, didn't find the image locally, pulled the image from a public registry called Docker Hub, created a container from that image, and ran the container for you.

> To generate this message, Docker took the following steps:
> 1. The Docker client contacted the Docker daemon.
> 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
> 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
> 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.


#### docker run 실행 결과 

Docker Clinet, Docker Daemon의 정체 

Docker daemon은 로컬에 받은 이미지가 있는지 확인하고 없으면 image를 검색해서 내려 받는다.

Docker daemon은 이미지를 실행가능한 상태로 만들기 위해 컨테이너를 생성하고 엔트리포인트를 실행한다. 그리고 출력값이 있으면 Docker client에게 전달한다.

`docker images`를 이용해서 실제로 없던 이미지를 받았는지 확인 가능 (from Docker Hub public registry)

`docker ps`, `docker ps -a`

`docker run --name [container-name] hello-world` --name 옵션을 이용해서 container 이름을 지정할 수도 있다 


## Build

### Dockerfile 작성 
```bash
$ cat > [파일이름] <<EOF
# 내용 입력 후 EOF
```
```bash
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF

# more at https://docs.docker.com/engine/reference/builder/#known-issues-run
```

`FROM` 베이스 부모 이미지, `WORKDIR` 컨테이너 안에서 작업할 디렉토리 선언, `ADD` 현재 디렉토리의 컨텐츠를 컨테이너 안에 어디로 옮길지 설정, `EXPOSE` 외부에서 컨테이너에 접속할 포트, `CMD` 컨테이너가 생성되서 첫 시작될 때 실행한 명령어

이렇게 작성된 Dockerfile를 가지고 Docker image를 만들 수 있다.

`docker build` 명령을 실행하면 Docker daemon이 Dockerfile에 정의된 내용을 따라서 Docker image를 만든다. 


### Application 작성 

```javascript
cat > app.js <<EOF
const http = require('http');

const hostname = '0.0.0.0';
const port = 80;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});

process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```

### Docker image 빌드 

```bash
$ docker build -t [image-name]:[tag] [Dockerfile 디렉토리]
$ docker images
```

## Docker container image 실행
```bash
$ docker run -p [host-port]:[container-port:80] --name [container-name] [image:tag]
$ curl http://localhost:4000
# backgound로 컨테이너를 실행하려면 -d 
# docker logs [container-id]

$ docker stop my-app && docker rm my-app # container 중지 && 삭제 
```

## Application 수정 

## 디버그 
```bash
$ docker logs -f [container-id]
$ docker exec -it [container-id] bash # 실행중인 container 안에서 작업하기, bash가 WORKDIR에서 실행된다. 
$ docker inspect [container-id] # container의 메타데이터 확인 
# more at https://docs.docker.com/engine/reference/commandline/inspect/#examples
# more at https://docs.docker.com/engine/reference/commandline/exec/
```

## 레지스트리에 Publish

Private 레지스트리에 업로드하기 위해서는 이미지 tag를 registry 이름으로 해야 한다.

[hostname]/[project-id]/[image]:[tag]

google container registry를 사용하는 경우 

hostname = gcr.io

gcloud config list project


다시 태그를 생성한다 
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2 

gcloud docker -- push gcr.io/[project-id]/[image]:[tag]

모든 컨테이너를 중지하고 지우는 방법 
> docker stop $(docker ps -q)

> docker rm $(docker ps -aq)


gcloud docker -- pull gcr.io/[project-id]/[image]:[tag]

docker run -p 4000:80 -d gcr.io/[project-id]/[image]:[tag]

curl http://localhost:4000

## 요약 - container의 portability
Docker 를 사용해서 public registry에서 베이스 이미지를 받아오고 새로운 이미지를 만들어서 public/private registry에 퍼블리시 할 수 있다. 그리고 그 이미지를 다시 받아서 어디서든지 이미지를 컨테이너로 실행할 수 있다. Host에 Docker를 설치하는 것을 제외하고는 Application 실행을 위한 dependency를 Host에 설치할 필요가 없다.

## More at

App Dev: Deploying the Application into Kubernetes Engine - Python
https://google.qwiklabs.com/focuses/1073?parent=catalog
Hello Node Kubernetes
https://google.qwiklabs.com/focuses/564?parent=catalog
Managing Deployments Using Kubernetes Engine
https://google.qwiklabs.com/focuses/639?parent=catalog