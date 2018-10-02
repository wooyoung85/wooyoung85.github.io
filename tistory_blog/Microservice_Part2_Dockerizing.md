> 마이크로 서비스 실습을 위해 시리즈로 Posting할 예정입니다. 
>  
> freeCodeCamp의 [Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882) 블로그 글을 토대로 작성하였습니다.
>
> 제 코드는 [Github](https://github.com/wooyoung85/kuber-sample-apps/tree/2.Dockerizing)에서 확인하실 수 있습니다.

# Container & Docker
## 1. 개념
### Container란?
> 전체 런타임 환경에서 애플리케이션과 종속 항목을 패키지화하고 <span style="color:#2496ed">격리</span>할 수 있도록 하는 기술

### Docker란?
> Container 기반의 오픈소스 가상화 플랫폼
- 구성요소
    - Docker Engine
    - Docker Image
    - Docker Container
    - Docker Client

## 2. 설치
### Mac
- [Docker Community Edition for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac)
- 설치파일로 Install

### Ubuntu
- [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- 설치 순서
    1. `apt update`
    2. 필요한 패키지를 먼저 설치  
    `sudo apt install -y apt-transport-https ca-certificates software-properties-common curl`
    3. Docker public키를 다운(유효한 설치파일인지 확인하기 위해)  
    `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
    4. apt-repository 설정  
    `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
    5. `apt update`
    6. Docker CE 버전 설치  
    `sudo apt install -y docker-ce`
    7. 확인  
    `docker`  
    `docker version`


## 3. 실습에 필요한 명령어
Docker 명령어가 예전과 좀 달라진 부분이 있음  
명령어가 너무 많아져서 복잡하기 때문에 명령어를 그룹화하여 제공함  
~~`docker pull alpine`~~  
~~`docker run alpine ls -l`~~

- 이미지  
    - 내려받기  
    `docker image pull alpine`  
    - 리스트 조회  
    `docker image ls`
    - 빌드하기  
    `docker image build -t wooyoung/hello-world`
    - repository에 올리기(기본적으로 Docker Hub에 올라가고, private한 repository는 전체경로를 입력해야함)  
    `docker image push wooyoung/hello-worlrd`
    - 지우기  
    `docker image rm <image ID>`
    - 모두 지우기  
    `docker image prune -a`


- 컨테이너  
    - 생성  
    `docker container run -d -p 8080:8080 --name say-hello hello-world:1`
    - 시작  
    `docker container start <container ID>`
    - 종료  
    `docker container stop <container ID>`
    - 실행  
    `docker container exec -it <container ID> sh`
    - 복사  
    `docker container cp index.html mynginx2:/usr/share/nginx/html/index.html`
    - 로그  
    `docker container logs <container ID>`
    - 지우기  
    `docker container rm <container ID>`
   
- 응용
    - Docker 이미지 리스트에서 ID값만 조회 후 xargs로 파라미터 전달하여 모든 이미지 지우기
    `docker image ls -q | xargs docker image rm`
    - Docker 컨테이너 전체 리스트에서 ID값만 조히 후 리눅스 환경변수로 전달하여 모든 컨테이너 정지하기  
    `docker container stop $(docker container ls -aq)`


# Dockerizing App
#### Docker Registry에 로그인
``` docker
docker login -u=<DOCKER_USER_ID> -p=<PASSWORD>
```
- 기본적으로 Docker Hub에 로그인 됨  
(private한 registry를 사용하려면 마지막에 서버 호스트 정보 입력해야 함)  
- Docker Agent 메뉴를 통해 로그인해도 됨

## Python Application
#### 0. 사전작업
```shell
$ pip freeze > requirements.txt
```

#### 1. Dockerfile 정의
##### Django Application Dockerfile
```docker
# Dockerfile
FROM python:3.6-slim

ENV CELERY_BROKER_URL redis://redis:6379/0
ENV CELERY_RESULT_BACKEND redis://redis:6379/0
ENV C_FORCE_ROOT true

ENV PROJECT_ROOT /app
WORKDIR $PROJECT_ROOT

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .

EXPOSE 8000
ENTRYPOINT ["python", "manage.py"]
CMD ["runserver", "0.0.0.0:8000"]
```

##### Celery Dockerfile
```docker
# Celery.Dockerfile
FROM python:3.6.4

ENV CELERY_BROKER_URL redis://redis:6379/0
ENV CELERY_RESULT_BACKEND redis://redis:6379/0
ENV C_FORCE_ROOT true

COPY . /app
WORKDIR /app

RUN pip install -r requirements.txt && \
    python -m textblob.download_corpora

ENTRYPOINT celery -A django_app worker --loglevel=info
```

#### 2. Docker Compose
docker-compose 명령어를 활용하여 동시에 Django, Celery, Redis 서비스 building & running
```docker
version: '3'

  services:
    django:
      build:
        context: .
        dockerfile: Dockerfile
      restart: always
      ports:
        - "8000:8000"
      depends_on:
        - redis
    worker:
      build:
        context: .
        dockerfile: Celery.Dockerfile
      depends_on:
        - redis
    redis:
      image: redis
```

## Spring Boot Application
#### 1. Dockerfile 정의
```docker
FROM openjdk:8-jdk-alpine
# Django Application url 환경변수로 정의
ENV SA_LOGIC_API_URL http://localhost:5000
ADD target/sentiment-analysis-web-0.0.1-SNAPSHOT.jar /
EXPOSE 8080
CMD ["java", "-jar", "sentiment-analysis-web-0.0.1-SNAPSHOT.jar", "--sa.logic.api.url=${SA_LOGIC_API_URL}"]
```

#### 2. Building & Pushing the container
```docker
# Building
$ docker build -f Dockerfile -t <DOCKER_USER_ID>/sentiment-analysis-web .

# 이미지 확인
$ docker image ls

# Pushing
$ docker push <DOCKER_USER_ID>/sentiment-analysis-web
```

#### 3. Running the container
```docker
$ docker run -d -p 8080:8080 <DOCKER_USER_ID>/sentiment-analysis-web
```

## React Application
#### 0. 사전 작업
아래 명령어를 실행하면 리액트 어플리케이션에 필요한 static 파일들을 모두 모아서 build 폴더에 넣어준다.
```shell
npm run build
```

```
# npm run build 후 프로젝트 폴더 구조
└─── react_app
    └─── build
    └─── node_modules
    └─── package.json
    └─── public
    └─── src
    └─── yarn.lock
```

#### 1. Dockerfile 정의
```docker
FROM nginx
COPY build /usr/share/nginx/html
```

#### 2. Building & Pushing the container
```docker
# Building
docker build -f Dockerfile -t <DOCKER_USER_ID>/sentiment-analysis-frontend .

# 이미지 확인
docker image ls

# Pushing
docker push <DOCKER_USER_ID>/sentiment-analysis-frontend
```
🙉 &nbsp;&nbsp;docker build 할 때 마지막에 꼭 `.` 을 찍어줘야 함!!

#### 3. Running the container
```docker
docker run -d -p 80:80 <DOCKER_USER_ID>/sentiment-analysis-frontend
```

## 참고자료
[Play with Docker classroom](https://training.play-with-docker.com/)

[![도커와 쿠버네티스, 두 마리 토끼를 잡자!](http://img.youtube.com/vi/Ajno86DrZv8/0.jpg)](https://www.youtube.com/watch?v=Ajno86DrZv8) 도커와 쿠버네티스, 두 마리 토끼를 잡자!

[Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882)