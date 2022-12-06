# WSL이란?

- 리눅스용 윈도우 하위 시스템(Windows Subsystem for Linux, `WSL`)

# WSL 설치

## WSL2 로 업그레이드

- Window Update 설치
- WSL2 활성화 명령어 실행
  ```shell
  # WSL feature Enable
  ps> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  # Virtual Machine Platform feature Enable
  ps> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```
  
  > 재부팅 필수!!!

## WSL 기본 설치 방법

- Microsoft Store 에서 Ubuntu 검색 후 설치
- `Ubuntu App`으로 실행할 수 있음
- Ubuntu 에 `root` 계정 설정

  ```bash
  $> sudo passwd root
  ```

- `root` 계정을 기본 사용자로 설정 (필수는 아님!!)

  > `PowerShell` 실행 후 아래 명령 실행

  ```shell
  ps> ubuntu config --default-user root

  # Ubuntu 세부 버전 지정해서 설치했다면
  ps> ubuntu1804.exe config --default-user root
  ```

## WSL 여러개 설치 (**이 방법을 추천!!!**)

> WSL을 여려개 설치할 경우 기본 사용자에 대한 설정이 필요함

- WSL 배포판 다운로드 [🚀 Link](https://docs.microsoft.com/ko-kr/windows/wsl/install-manual#downloading-distributions)
- 다운받은`appx` 파일을 `zip` 파일로 변경 후 압축해제  
  (Ubuntu_2004.2020.424.0_x64.appx 👉 Ubuntu_2004.2020.424.0_x64.zip)
- WSL 관리 폴더 생성
  ```shell
  ps> mkdir -p E:\WSL\DATA
  ```
- Import & Distribution
  ```shell
  # wsl --import <배포> <설치 위치> <파일 이름>
  ps> wsl --import Ubuntu-1 E:\WSL\DATA\Ubuntu-1 E:\Ubuntu_2004.2020.424.0_x64\install.tar.gz
  # Ubuntu 실행
  ps> wsl --distribution Ubuntu-1
  ```
- Ubuntu 사용자 추가

  ```bash
  # 패스워드 설정 후 기본 정보 입력은 그냥 엔터 누르면 됨
  $> adduser wooyoung
  $> usermod -aG sudo wooyoung
  # WSL 기본 사용자 지정(WSL 재부팅 해야 적용됨!)
  $> cat <<EOF > /etc/wsl.conf
  [user]
  default=wooyoung
  EOF
  $> exit
  ```

- WSL 재부팅

  ```shell
  ps> wsl -t Ubuntu-1
  ```

- WSL 관련 명령어
  ```shell
  # WSL 버전 확인 (전부 버전 2 로 확인되면 아래 내용은 Pass!)
  ps> wsl --list --verbose
  # Ubuntu Wsl 버전 변경
  ps> wsl --set-version Ubuntu 2
  # WSL 2를 기본 버전으로 설정
  ps> wsl --set-default-version 2
  # 사용자 지정해서 WSL 실행
  ps> wsl --distribution Ubuntu --user wooyoung
  # WSL 삭제
  ps> wsl --unregister Ubuntu
  ```

# Ubuntu 사용

## WSL 실행

- Windows Terminal 로 실행 (**이 방법을 추천!!!**)

  <img src="./images/Tools/wsl-windows-terminal.png" width="500">

- Powershell 에서 실행하기

  ```shell
  ps> wsl -d Ubuntu-1 -u wooyoung --cd ~
  ```

## Ubuntu 환경 설정

- Ubuntu 기본 쉘 변경 (`dash` 👉 `bash`)

  ```bash
  # 아래 명령어 입력 후 나오는 화면에서 No 라고 답변
  $> sudo dpkg-reconfigure dash
  # 확인
  $> ls -al /bin/sh
  lrwxrwxrwx 1 root root 4 Jun 18 12:41 /bin/sh -> bash
  ```

  <img src="./images/Tools/wsl-shell-change.png" width="640">

- apt 주소 변경 (가장 가까운 서버에 접근하도록)

  ```bash
  $> sudo sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
  $> sudo sed -i 's/security.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
  ```

- 패키지 정보 업데이트 및 설치된 패키지 최신화
  ```bash
  $> sudo apt-get -y update && sudo apt-get -y upgrade
  $> sudo apt-get -y full-upgrade
  ```

## zsh 설정

- zsh 설치

  ```bash
  $> sudo apt-get install -y zsh
  ```

- `Oh-my-zsh`

  ```bash
  # Oh My Zsh 설치
  $> sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  # powerlevel10k 테마 설정
  $> git clone https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/themes/powerlevel10k
  $> sed -i 's/ZSH_THEME=\"robbyrussell\"/ZSH_THEME=\"powerlevel10k\/powerlevel10k\"/g' ~/.zshrc
  $> exec zsh
  ```

- Font 깨짐

  - [https://www.nerdfonts.com/font-downloads](https://www.nerdfonts.com/font-downloads) 에서 원하는 폰트 다운로드 후 Windows Terminal 에 폰트 설정
  - `JetBrainsMono Nerd Font` 추천

# WSL 네트워크

## 네트워크 구성

- Host PC

  ```shell
  PS C:\Users\wooyoung> ipconfig
  ...
  이더넷 어댑터 vEthernet (WSL):

    연결별 DNS 접미사. . . . :
    링크-로컬 IPv6 주소 . . . . : fe80::1149:7e0e:47e3:b275%42
    IPv4 주소 . . . . . . . . . : 172.29.176.1
    서브넷 마스크 . . . . . . . : 255.255.240.0
    기본 게이트웨이 . . . . . . :
  ```

- WSL
  ```bash
  $> ifconfig
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.29.191.98  netmask 255.255.240.0  broadcast 172.29.191.255
          inet6 fe80::215:5dff:febf:b131  prefixlen 64  scopeid 0x20<link>
          ether 00:15:5d:bf:b1:31  txqueuelen 1000  (Ethernet)
          RX packets 348639  bytes 101575823 (101.5 MB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 29001  bytes 2561521 (2.5 MB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  ```

## 네트워크 특징

- (Host => WSL) WSL 에서 `8080` 포트로 웹서비스를 실행하면 Host PC에서 `localhost:8080` 으로 접근 가능
- (WSL => Host) Host에서 `3306` 포트로 DB가 실행 중이라면 WSL에서 `172.29.176.1:3306` 으로 접근 가능

## WSL 외부 네트워크와 연결하기

- Host PC에 포트 포워딩 설정 추가

  ```shell
  PS C:\Users\wooyoung> netsh interface portproxy add v4tov4 listenport=15021 listenaddress=0.0.0.0 connectport=21 connectaddress=172.29.191.98
  # 포트포워딩 설정 확인
  PS C:\Users\wooyoung> netsh interface portproxy show v4tov4
  ipv4 수신 대기:             ipv4에 연결:

  주소            포트        주소            포트
  --------------- ----------  --------------- ----------
  0.0.0.0         15021       172.29.191.98   21

  # 포트포워딩 설정 삭제
  PS C:\Users\wooyoung> netsh interface portproxy delete v4tov4 listenport=15021 listenaddress=0.0.0.0
  ```

## 참고자료

[Linux용 Windows 하위 시스템 설명서](https://docs.microsoft.com/ko-kr/windows/wsl/)  
[ZSH 설정 소개](https://black7375.tistory.com/59)  
[WSL 2의 네트워크 통신 방법](https://www.sysnet.pe.kr/2/0/12347)
