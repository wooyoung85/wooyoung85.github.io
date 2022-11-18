> 윈도우 WSL Ubuntu 환경에서 진행하였습니다.

# gcloud CLI 설치 (Ubuntu)

```bash
sudo apt-get install apt-transport-https ca-certificates gnupg
# 패키지 소스로 gcloud CLI 배포 URI 추가
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
# Google Cloud 공개 키를 가져오기
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
# gcloud CLI를 업데이트하고 설치
sudo apt-get update && sudo apt-get install google-cloud-cli
```

# gcloud CLI 사용하기

> 둘 중 하나만 하면됨

- gcloud 초기화

  ```bash
  gcloud init
  ```

- 로그인 및 프로젝트 설정
  ```bash
  gcloud auth login --no-launch-browser
  gcloud config set project <PROJECT_ID>
  ```

# Compute Engine VM Instance

- 환경변수

  ```bash
  PROJECT=$(gcloud config get-value project)
  ACCOUNT=$(gcloud config get-value account)
  ZONE=asia-northeast3-a
  ```

  > Seoul 리전은 `asia-northeast3-a`

- VM 이미지 검색

  ```bash
  gcloud compute images list --project ubuntu-os-cloud --no-standard-images
  ```

- VM 생성

  ```bash
  gcloud compute instances create master worker1 worker2 \
  --zone=$ZONE \
  --custom-cpu=2 \
  --custom-memory=4 \
  --image-family=ubuntu-2004-lts \
  --image-project=ubuntu-os-cloud  \
  --boot-disk-size=50GB
  ```

# VM SSH 접속

- 서버 접속 시 사용할 SSH Key 생성

  ```bash
  ssh-keygen -m PEM -t rsa -b 4096 -C $ACCOUNT -f ~/.ssh/gcp_rsa
  chmod 400 ~/.ssh/gcp_rsa
  ```

- Compute Engine 에 SSH Key 등록

  ```bash
  gcloud compute config-ssh --ssh-key-file=~/.ssh/gcp_rsa
  ```

- SSH 접속
  ```bash
  ssh master.$ZONE.$PROJECT
  ssh worker1.$ZONE.$PROJECT
  ssh worker2.$ZONE.$PROJECT
  ```
