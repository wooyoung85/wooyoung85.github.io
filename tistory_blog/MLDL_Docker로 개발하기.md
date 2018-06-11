## Docker 사용 이유
- Docker를 이용하면 OS 또는 애플리케이션 구축에 드는 번거로움을 덜고, 쉽게 개발 환경 구축이 가능
- PC에 직접 여러 프로그램을 설치하는 것이 아니라 Docker 이미지를 가져다가 쉽게 구축 가능
```docker
// miniconda 이미지 가져오기
docker pull continuumio/miniconda3

// miniconda 이미지로 Docker 컨테이너 실행
docker run -i -t continuumio/miniconda3 /bin/bash
```

## Docker 설치
- Docker를 사용하기 위해서는 Mac OS는 Yosemite(10.10.3), Windows는 10 Pro(64bit) 이상의 환경이 필요
(Windows 7에서는 Docker Toolbox를 통해 실습 가능하지만 추천하고 싶지 않습니다)

## Docker 설정
- Not enough memory to start Docker 오류 발생 시
    - 우측하단 Docker 트레이 아이콘 우클릭
    - Settings
    - Advanced탭에서 Docker에 할당할 CPU, Memory 자원 조절


- <code>Docker run -v</code> 옵션으로 로컬 PC 폴더 마운트 할 때 오류 발생 시
    - 우측하단 Docker 트레이 아이콘 우클릭
    - Settings
    - Shared Drive탭에서 Docker에서 Mount 폴더로 사용할 드라이브 체크