---
layout: post
title: "Docker로 ASP.NET Core Web API 배포하기"
description: "Dockerize ASP.NET Core Web API Application"
date: 2017-06-12
comments: true
share: true
---

이번 포스팅에서는 Docker 컨테이너에 ASP.NET Web API Application을 배포하는 실습을 해보겠습니다.

진행하기 전에 PC환경을 간단하게 설명드리면
> OS: Windows 7 64bit <sup>(Windows 10이면 더 좋을 것 같고, Docker를 사용하시려면 64bit 운영체제를 사용해야만 합니다)</sup><br>
> [Docker Toolbox 설치](https://www.docker.com/products/docker-toolbox)

## Project 생성
###### 저는 Visual Studio 2017 Community 버전을 사용하고 있습니다.
[마이크로소프트에서 제공하고 있는 ASP.NET Tutorial](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api)을 바탕으로 Web API프로젝트를 구성하겠습니다.

![image_1](/images/post_2/1.png)
- 파일 > 새로만들기 > 프로젝트
- 템플릿 중 ASP.NET Core 웹 응용 프로그램 (.NET Core) 선택