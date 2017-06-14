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
> OS는 Windows 7 64bit (Windows 10이면 더 좋을 것 같고, Docker를 사용하시려면 64bit 운영체제는 필수입니다)<br>
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
- Docker Quickstart Terminal을 실행하면 가상머신을 생성(최초에 한 번)하고 도커 터미널이 실행된다.<br>
![image_2](/images/post_3/2.png)


