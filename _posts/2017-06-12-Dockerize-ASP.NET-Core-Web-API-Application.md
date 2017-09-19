---
layout: post
title: "Docker로 ASP.NET Core Web API 배포하기"
description: "Dockerize ASP.NET Core Web API Application"
date: 2017-06-12
tags: [Docker, ASP.NET Web API, DockerFile]
comments: true
share: true

sitemap :
  changefreq : weekly
  priority : 1.0
---

이번 포스팅에서는 Docker 컨테이너에 ASP.NET Web API Application을 배포하는 방법에 대해 써보려고 합니다.

#### 운영체제 및 사전 준비 환경
> OS는 Windows 7 64bit (Docker를 사용하시려면 64bit 운영체제는 필수입니다)<br>
> BIOS 설정에서 PC CPU 가상화 지원 기능 Enabled로 변경<br>
> [Window용 Docker Toolbox 설치](https://www.docker.com/products/docker-toolbox)
(Git은 꼭 설치되어야 합니다)

만약 Docker를 처음 접하시는 분들이라면 이 동영상을 보시면 많은 도움이 될 것 같습니다~<br>
[![도커 학습과 Boot2Docker](http://img.youtube.com/vi/MqL5exxZDg4/0.jpg)](https://youtu.be/MqL5exxZDg4) 

## Docker Toolbox 둘러보기
Toolbox 설치를 완료하면 아래 3가지 아이콘이 보이게됩니다.<br>
![image_1](/images/post_3/1.png)<br>
<sup>(Docker Quickstart Terminal, Oracle VM VirtualBox, Kitematic(Alpha))</sup>

- 먼저 Oracle VM VirtualBox의 버전을 최신화 해주는 것이 좋습니다.(필수 X))
- Kitematic은 Docker를 좀 더 편하게 사용하기 위한 GUI 툴
- Docker Quickstart Terminal을 실행하면 가상머신을 생성(최초에 한 번)하고 도커 터미널이 실행됩니다.  (Default로 생성한 가상머신에 연결된 상태)<br>
![image_2](/images/post_3/2.png)
- Default 가상머신 IP는 Docker Container에 접근할 때 필요하기 때문에 잘 기억해 두어야 합니다.<br>
(혹시 까먹었다면  `docker-machine ip` 로 확인 가능)

## 빠르게 Docker 컨테이너에 ASP.NET Core Web Application 실행하기
#### .NET Core 설치된 컨테이너 실행
~~~
$ docker run -it --name aspnetcore -p 5050:5000 microsoft/dotnet:latest
~~~
> <b>run</b> : 이미지를 컨테이너로 실행 (run 명령어 실행 시 이미지를 내려받은 적이 없다면 이미지를 받은 후 컨테이너 실행)<br> 
> <b>-i(interactive), -t(Pseudo-tty)</b> : run 명령어 옵션, 실행된 컨테이너에 bash 명령을 사용할 수 있습니다.<br> 
> <b>--name</b> : 컨테이너의 이름을 지정<br>
> <b>-p</b> : 포트 설정 (Host Port:Docker Container Port)

##### Docker Image의 Tag는 프로젝트 생성 시 사용한 .NET Core 버전을 잘 확인해서 설정하는 것이 좋습니다.<br><sup>ex) .NET Core 1.1을 사용 시 → dotnet:1.1-sdk-projectjson</sup>
##### 최신버전이라고 마냥 좋은 건 아니고 환경에 맞게 [도커허브 microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/) 사이트를 참고해서 설정하세요~

![image_3](/images/post_3/3.png)
- 명령어를 실행하면 이미지를 다운(최초 한 번만) 받고 컨테이너에 접속

<span style='color:orange'>(참고)</span><br>
`exit` 를 쳐서 빠져나갈 수 있는데, 이렇게 되면 컨테이너 실행이 중지된다. `Ctrl + p,q` 로 빠져나와야 컨테이너를 중지하지 않고 빠져나올 수 있다.

#### .NET Core Web Application 생성과 실행
~~~
mkdir myapp
cd myapp
~~~
> - myapp 폴더 생성 후 이동

~~~
dotnet new web
~~~
> - Web 프로젝트 생성

<span style='color:red'>**(중요)**</span><br>
기본적으로 Kestrel(cross-platform web server for ASP.NET Core )은 `localhost`(loopback interface)에서 수신 대기하도록 설정되어 있어서 .Net Application이 Docker Container 안에서 실행되는 경우 컨테이너 내부에서만 접근이 가능해진다.<br><br>
이에 대한 해결책은 Program.cs에서 모든 IP Address에 대해 Listening 할 수 있도록 설정해 줘야 한다.

~~~ diff
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseKestrel()
        .UseContentRoot(Directory.GetCurrentDirectory())
+        .UseUrls("http://*:5000")
        .UseIISIntegration()
        .UseStartup<Startup>()
        .Build();

    host.Run();
}
~~~
<span style='color:orange'>(참고)</span>
<pre>
  apt-get update
  apt-get install vim
  vim Program.cs
</pre>

- 위 명령어를 통해 Docker Container에 vim 편집기를 설치하고 Program.cs 파일을 수정
- vim 편집기 사용법은 구글링하면 많이 나옵니다~^^
~~~
dotnet restore
dotnet run
~~~
- Defendency 복원 (.NET Core에서는 Restore 명령어를 통해 프로젝트에 종속된 패키지를 복원함)<br>
- 웹사이트 실행

![image_4](/images/post_3/4.png)
- 웹사이트 실행이 성공하면 에러메세지 없이 수신 대기 중인 IP Address와 Port 번호가 보이게 됨<br>
`Now listening on : http://*:5000` 

#### 테스트
![image_5](/images/post_3/5.png)
- 가상머신의 IP(192.168.99.100)와 컨테이너의 5000 Port와 Mapping 된 5050 Port로 접속

## 간편한 배포를 위한 DockerFile 작성하기
DockerFile을 활용해서 배포를 하려면 아래와 같은 순서로 진행하면 된다.
> 1. 가상머신에 GitHub 저장소 복제 
> 2. 소스코드가 있는 폴더에서 <b>DockerFile</b> 생성
> 3. DockerFile <b>build</b>해서 이미지 생성
> 4. 생성된 이미지를 사용하여 Docker 컨테이너 <b>run</b>

#### GitHub 저장소 복제
~~~
 git clone https://github.com/wooyoung85/TodoApi.git
 cd TodoApi/TodoApi
~~~
- 가상머신에 저장소 복제 후 소스코드가 있는 폴더로 이동

#### DockerFile 만들기
~~~ dockerfile
FROM microsoft/dotnet:1.1-sdk-projectjson

COPY . /myapp
WORKDIR /myapp

RUN ["dotnet", "restore"]
RUN ["dotnet", "build"]

EXPOSE 5000/tcp
ENV ASPNETCORE_URLS http://*:5000

ENTRYPOINT ["dotnet", "run"]
~~~
- dotnet 설치는 microsoft/dotnet:1.1-sdk-projectjson 이미지를 사용
- 컨테이너 내부에 애플리케이션 소스를 myapp폴더에 복사
- myapp폴더를 작업 폴더로 설정
- Defendency 복원하고 빌드
- 가상머신에서 5000 포트를 노출시키도록 설정
- 컨테이너에서 실행될 .Net Application에 환경 변수(ASPNETCORE_URLS)를 설정
- 컨테이너가 시작되면 dotnet run명령 이 실행됨

#### DockerFile 빌드하기
~~~ docker
$ docker build -t wooyoung85/todoapi .
~~~
- 이미지를 생성하려면 Dockerfile을 사용하여 같은 폴더에서 명령을 실행

#### 생성된 Image 사용하여 Docker RUN
~~~ docker
$ docker run -it wooyoung85/todoapi --name todoapi
~~~
- 컨테이너가 실행되면서 어플리케이션도 동시에 실행됨
- http://192.168.99.100:5000/api/Todo 로 테스트

<br>
---
**참고한 사이트** <br>
[HOSTING .NET CORE ON LINUX WITH DOCKER - A NOOB'S GUIDE](http://blog.scottlogic.com/2016/09/05/hosting-netcore-on-linux-with-docker.html)<br>
[DOCKERIZING AN ASP.NET CORE APPLICATION WITH GITHUB, DOCKER CLOUD AND AZURE](https://radu-matei.github.io/blog/aspnet-core-docker-azure/)<br>
