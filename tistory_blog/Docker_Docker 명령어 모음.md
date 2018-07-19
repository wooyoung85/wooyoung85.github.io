## Docker 명령어 모음

자주 쓰는데 맨날 헷갈리는 명령어만 모아놨습니다ㅎㅎ

### 이미지 검색

```
docker search [image명]
```

### 이미지 내려받기

```
docker pull ubuntu:latest
```

### 이미지 목록 보기

```
docker images
```
 

### 컨테이너 리스트 조회 (-a : 정지된 컨테이너 리스트도  포함)

```
docker ps -a
```

### 컨테이너 시작

```
docker start <container id/name>
```

### 컨테이너 접속

```
docker attach <container id/name>
```

### 컨테이너 삭제 (-f : 강제로 삭제)

```
docker rm <container id/name>
```

### 컨테이너 시작+접속

```
docker run -i -t -v /c/Users/awood/sample:/sample --name mlearn  mlearn:init /bin/bash

-v : 폴더 마운트
--name : 컨테이너 이름 설정
```

**※ 컨테이너에 접속 후 CTRL-p CTRL-q 로 빠져나오기!**