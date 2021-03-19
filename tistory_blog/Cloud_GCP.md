# Google Cloud Platform 사용하기

## 프로젝트 생성

VM 인스턴스를 생성하려면 프로젝트가 있어야 함
![프로젝트 생성](./images/Cloud/1.png)

## cloud shell 활성화

![cloud shell 활성화](./images/Cloud/2.png)

- cloud shell 활성화한 후 프로젝트가 잘 설정되었는지 꼭 확인해야 함
- `gcloud config set project [PROJECT_ID]` 로 설정 가능

## starup-script 준비

- VM 인스턴스를 생성하면 기본적으로 password를 통한 ssh 접속을 막아두고 있기 때문에 아래와 같이 설정하여 root 계정으로 ssh 접속을 하려고 함

  > 🥳**updated** 🥳  
  > 정상적으로 접속하는 방법은 [Google Cloud Platform VM 접속하기](https://wooyoung85.tistory.com/59) 를 참고하시기 바랍니다.

- cloudshell에 `install.sh` 파일 생성 후 아래 내용 복사 붙여넣기 (**CentOS6 용**)

  ```bash
  #!/bin/bash
  sudo yum -y install expect

  expect -c "set timeout 10
  spawn sudo passwd
  expect \"New password:\"
  send \"password\r\"
  expect \"Retype new password:\"
  send \"password\r\"
  expect eof"

  sudo sed -i 's/^PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config
  sudo sed -i "s/^PasswordAuthentication .*/PasswordAuthentication yes/g" /etc/ssh/sshd_config

  sudo service sshd restart
  ```

  (**🙉주의사항 : 이 스크립트를 그대로 사용하면 root계정의 비밀번호는 password로 설정됨!!! 꼭 바꿔서 사용하세요~~**)

- **Ubuntu 16.04 OS** 사용 시 아래 스크립트 사용

  ```bash
  #!/bin/bash
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get -y install expect

  expect -c "set timeout 10
  spawn sudo passwd
  expect \"Enter new UNIX password:\"
  send \"root123\!\r\"
  expect \"Retype new UNIX password:\"
  send \"root123\!\r\"
  expect eof"

  sudo sed -i 's/^PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config
  sudo sed -i "s/^PasswordAuthentication .*/PasswordAuthentication yes/g" /etc/ssh/sshd_config

  sudo systemctl restart networking
  ```

## VM 인스턴스 생성

- [Cloud SDK - gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) 문서에 인스턴스 생성하는 명령어가 아주 자세하게 나와있음
- 머신유형을 Customizing 하고 부팅 디스크 이미지와 용량을 변경하는 스크립트 예제
- 머신유형을 Customizing하려면 `machine-type` flag 대신 `custom-cpu` 와 `custom-memory`를 함께 설정해 주어야 함
  ```bash
  gcloud compute instances create node1 \
  --zone=asia-northeast1-a \
  --custom-cpu=2 \
  --custom-memory=12 \
  --image-family=centos-6 \
  --image-project=centos-cloud  \
  --boot-disk-size=100GB \
  --metadata-from-file startup-script=install.sh
  ```
- 머신을 동일한 설정으로 여러대 띄우기
  ```bash
  gcloud compute instances create node2 node3 node4 node5 \
  --zone=asia-northeast1-a \
  --machine-type=n1-standard-1 \
  --image-family=centos-6 \
  --image-project=centos-cloud  \
  --boot-disk-size=100GB \
  --metadata-from-file startup-script=install.sh
  ```

## 방화벽 설정

- 왼쪽 상단 메뉴에서 `VPC 네트워크 >> 방화벽 규칙` 으로 가면 설정 가능
  ![](./images/Cloud/3.png)

- 필요한 사항을 잘 설정하면 됨  
  ![](./images/Cloud/4.png)
  > 소스 IP범위를 `0.0.0.0/0`으로 하면 누구나 접근이 가능하기 때문에 DDOS 공격의 우려가 있습니다~!! 이점 꼭 유의하시기 바랍니다 ^^

## 내 IP 확인 방법

- 소스 IP를 설정하려면 내 IP를 확인해야하는데 [myipaddress.com](http://www.myipaddress.com) 에 접속하면 내 IP를 바로 확인할 수 있음
- class는 아래 규칙에 맞게 설정해 주면 됨

  | Class   | 첫 번째 Octet 범위                       |
  | ------- | ---------------------------------------- |
  | A Class | 0 ~ 127 (0.0.0.0 ~ 127.255.255.255)      |
  | B Class | 128 ~ 191 (128.0.0.0 ~ 191.255.255.255)  |
  | C Class | 192 ~ 223 (192.0.0.0 ~ 233.255.255.255)  |
  | D Class | 224 ~ 239 (224.0.0.0 ~ 239.255.255.255)  |
  | E Class | 240 ~ 255 (240.0.0.0. ~ 255.255.255.255) |

> 만약에 내 IP가 211.10.9.8 이라고 가정해 본다면 C클래스이기 때문에 방화벽 설정 시 소스 IP범위를 `211.10.9.0/24`로 설정하면 된다.

## 참고 자료

[Cloud SDK - gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)  
[IP 주소체계와 클래스 구별법 (IPV4)](http://korean-daeddo.blogspot.com/2015/12/ip.html)
