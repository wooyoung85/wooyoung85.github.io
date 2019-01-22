# Virtual Box를 활용한 Spark 실습 환경 구축

## 1. Virtual Box 설치
5.22 버전 사용 <sup>(최신버전은 6.0)</sup>  
[https://www.virtualbox.org/wiki/Download_Old_Builds_5_2](https://www.virtualbox.org/wiki/Download_Old_Builds_5_2)


## 2. 가상머신 생성 (Linux, RedHat64bit)
- CentOS7을 사용할 예정
- VirtualBox 관리자 새로만들기에서 아래와 같이 설정
- 이름을 알아서 적절하게 ^^


## 3. VirtualBox 네트워크 구성
#### NAT네트워크 추가
- 공유기와 유사한 환경
- NAT네트워크(공유기)에 연결된 VM들이 하나의 네트워크상에서 동작
- 내부 VM들 간 통신 O, 외부 시스템에서는 직접 내부에 접근 X

> 설정(`cmd`+`,`) > 네트워크 > 추가

#### 호스트 네트워크 추가
- 호스트(내PC)에서 VM에 접근하기 위한 설정
- VM에 고정 IP를 설정한 후 ssh 접속
> 파일 (virtual box관리자 포커스) > 호스트 네트워크 관리자 > 만들기

#### VM 네트워크 어댑터 설정
- VM 설정 > 네트워크 Tab
- 아래와 같이 설정 추가


## 4. CentOS7 설치
- ISO 파일 다운로드 (Minimal Version도 가능)  
[https://www.centos.org/download/](https://www.centos.org/download/)

- CentOS 이미지 삽입
- VM 시작

#### 설치 시 주의사항
- 파티션 지정
- 네트워크 켬 enp0s3, enp0s8 활성화


## 5. 재부팅


## 6. 네트워크 설정
#### 네트워크 어댑터 확인

```shell
ip addr
```

#### 고정 IP 잡기

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0s8
```
#### 아래 내용 추가 및 수정
>IPADDR=192.168.56.XXX  
NETMASK=255.255.255.0  
ONBOOT=yes  
BOOTPROTO=static

#### 네트워크 서비스 재시작

```shell
service network restart
```


## 7. ssh 접속
```shell
ssh root@192.168.56.XXX
```

## 8. CentOS 기본 라이브러리 설치
```shell
yum update
yum groupinstall 'Development Tools'
```

## 9. 자바 설치


## 10. 스파크 설치


## 11. 방화벽 해제 설정
#### 상태 확인
```shell
systemctl status firewalld
systemctl status iptables
systemctl status ip6tables
```
#### stop & disable
```shell
systemctl stop firewalld
systemctl stop iptables
systemctl stop ip6tables

systemctl disable firewalld
systemctl disable iptables
systemctl disable ip6tables
```