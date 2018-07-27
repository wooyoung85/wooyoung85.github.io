스프링 부트 레퍼런스 문서를 보면서 레디스 실습을 하려고 오랜만에 도커 환경을 사용했는데 docker 이미지가 제대로 받아지지가 않는 것 같았다.

## 해결방법
- 찾아보니 네트워크 방화벽을 확인해보라는 얘기가 많았고, 재설치 하라는 솔루션도 있었다.ㅋ
- 하지만 나는 Docker 재부팅으로 해결했다.

1. 우측 하단의 Docker 트레이 아이콘 우클릭 > Settings 메뉴 선택
2. Reset > Restart Docker

- Docker 재부팅 후 다시 `docker pull` 했더니 잘된다~^^

## 참고자료
[Docker - GitHub Issues](https://github.com/docker/for-mac/issues/1317)