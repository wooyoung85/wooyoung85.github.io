---
layout: post
title: "Docker로 ASP.NET Core Web API 배포하기"
description: "Dockerize ASP.NET Core Web API Application"
date: 2017-06-12
comments: true
share: true

sitemap :
  changefreq : weekly
  priority : 1.0
---

이번 포스팅에서는 Docker 컨테이너에 ASP.NET Web API Application을 배포하는 방법에 대해 써보려고 합니다.

#### 운영체제 및 사전 준비 환경
> OS는 Windows 7 64bit <sup>(Windows 10이면 더 좋을 것 같고, Docker를 사용하시려면 64bit 운영체제는 필수입니다)</sup><br>
> BIOS 설정에서 PC CPU 가상화 지원 기능 Enabled로 변경<br>
> [Window용 Docker Toolbox 설치](https://www.docker.com/products/docker-toolbox)
##### Toolbox 설치 관련해서 구글링하면 많은 블로그들이 있으니 설치는 알아서 열심히 하는걸로...

Docker를 처음 접하시는 분들이라면 이 동영상을 보시면 많은 도움이 될 것 같습니다~<br>
![Video Label](https://www.youtube.com/watch?v=MqL5exxZDg4) 

## Docker Toolbox 
1. Docker Quickstart Terminal
2. Kitematic(Alpha)
3. Oracle VM VirtualBox

