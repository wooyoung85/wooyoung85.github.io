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

- Interactive Shell 사용하기
  ```bash
  # gcloud beta 가 설치되어 있는지 확인
  gcloud components list --filter="id:beta"
  # gcloud beta 설치
  ## 이미 설치되어 있는데 또 설치하면 에러남😣
  gcloud components install beta
  # Interactive 모드로 전환
  gcloud beta interactive
  ```

# 프로젝트

- 프로젝트 생성

  ```bash
  PROJECT_ID=woodong-prj-$(date +%Y%m%d)
  gcloud projects create $PROJECT_ID --name="Test Project"
  gcloud config set project $PROJECT_ID
  ```

  > 프로젝트는 console 화면에서 진행하는 것을 권장드립니다.  
  > `gloud cli`로 생성한 프로젝트가 바로 반영이 안되거나 결제 정보가 안 물리는 등의 문제가 발생했습니다 ㅠ

- 환경변수

  ```bash
  PROJECT=$(gcloud config get-value project)
  ACCOUNT=$(gcloud config get-value account)
  ZONE=asia-northeast3-a
  echo $PROJECT
  echo $ACCOUNT
  echo $ZONE
  ```

  > Seoul 리전은 `asia-northeast3-a`

# Compute Engine VM Instance

- Compute Engine API 활성화

  ```bash
  gcloud services enable compute.googleapis.com
  ```

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

  # gcloud compute instances delete master worker1 worker2
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
  > 인스턴스 생성이 완료된 후 접속 시도해야 함

# multi-node Kubernetes cluster using the `kubeadm` tool(1.24)

## 1. Installing kubeadm: 사전 준비 : All Nodes

- system 구성 확인 및 설정

  ```bash
  sudo -i

  cat /etc/os-release
  lscpu
  free -h
  hostname
  ip addr

  cat <<EOF >> /etc/hosts

  10.178.0.2 master
  10.178.0.3 worker1
  10.178.0.4 worker2
  EOF

  cat /etc/hosts

  ufw status
  swapon -s
  ```

- IPv4를 포워딩하여 iptables가 bridge된 traffic 활성화

```bash
# br_netfilter 모듈을 로드
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# bridge taffic 보게 커널 파라메터 수정
# 필요한 sysctl 파라미터를 /etc/sysctl.d/conf 파일에 설정하면, 재부팅 후에도 값이 유지된다.
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 재부팅하지 않고 sysctl 파라미터 적용하기
sysctl --system
```

## 2. Installing a container runtime : All Nodes

- 1. **docker 엔진 설치**
  ```bash
  apt update
  apt install -y docker.io
  systemctl enable --now docker
  systemctl status docker
  ```
- 2. cre-docker install
  - **cri-docker 엔진 설치**
    ```bash
    wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.6/cri-dockerd_0.2.6.3-0.ubuntu-bionic_amd64.deb
    dpkg -i cri-dockerd_0.2.6.3-0.ubuntu-bionic_amd64.deb
    systemctl status cri-docker
    ```

## 3. Installing kubeadm : ALL

- 모든 머신에 다음 패키지들을 설치한다.

  - `kubeadm`: 클러스터를 부트스트랩하는 명령이다.
  - `kubelet`: 클러스터의 모든 머신에서 실행되는 파드와 컨테이너 시작과 같은 작업을 수행하는 컴포넌트이다.
  - `kubectl`: 클러스터와 통신하기 위한 커맨드 라인 유틸리티이다.
  - Installing kubeadm, kubelet and kubectl : master, node1, node2

    ```bash
    # Update the apt package index and install packages needed to use the Kubernetes apt repository:
    apt-get update
    apt-get install -y apt-transport-https ca-certificates curl
    # Download the Google Cloud public signing key:
    curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    #Add the Kubernetes apt repository:
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list &&
    #Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
    sudo apt-get update
    sudo apt-get install -y kubelet=1.24.0-00 kubeadm=1.24.0-00 kubectl=1.24.0-00 &&
    sudo apt-mark hold kubelet kubeadm kubectl

    kubeadm version
    kubelet --version
    kubectl version --client
    ```

## 4. Creating a cluster with kubeadm

- Initializing your `control-plane` node : master 에서만 실행

  ```bash
  # control-plaine 컴포넌트 구성
  kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
  ...
  [addons] Applied essential addon: kube-proxy

  Your Kubernetes control-plane has initialized successfully!

  # Kubectl을 명령 실행 허용하려면 kubeadm init 명령의 실행결과 나온 내용을 동작해야 함
  mkdir -p $HOME/.kube
  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  chown $(id -u):$(id -g) $HOME/.kube/config

  # root 권한으로 worker node join을 위한 명령어. 별도 저장해둠.
  cat <<EOF > token.join
  kubeadm join 10.178.0.2:6443 --token s2i7yt.bzjo664xm09wpncw \
        --discovery-token-ca-cert-hash sha256:18c00f5e46273a8465aa9ecd977373275e4669636975b5df2528fb485047baae
    --cri-socket unix:///var/run/cri-dockerd.sock
  EOF
  ```

- Installing a Pod network add-on

  - Calico

    - install

      ```bash
      # CNI(컨테이너 네트워크 인터페이스) 기반 Pod 네트워크 추가 기능을 배포해야 Pod가 서로 통신할 수 있습니다. 네트워크를 설치하기 전에 클러스터 DNS(CoreDNS)가 시작되지 않습니다.
      kubectl get nodes
      NAME       STATUS     ROLES           AGE     VERSION
      master     NotReady   control-plane   7m31s   v1.24.8

      # Pod 네트워크가 호스트 네트워크와 겹치지 않도록 주의해야함.
      # Calico 설치
      **kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
      kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml

      #** node 초기화 될때까지 기다림
      **watch kubectl get pods -n calico-system

      kubectl get pods -n calico-system**
      NAME                                       READY   STATUS    RESTARTS   AGE
      calico-kube-controllers-85666c5b94-927wg   1/1     Running   0          82s
      calico-node-hcnp9                          1/1     Running   0          82s
      calico-typha-f8845f557-cwgmx               1/1     Running   0          83s
      csi-node-driver-l94kh                      2/2     Running   0          40s

      ****# 다시 확인
      **kubectl get nodes**
      NAME     STATUS   ROLES           AGE     VERSION
      master   Ready    control-plane   5m15s   v1.24.8

      # Calico Mode 변경- 지왕님 search 도움(고마워요 지왕님)
      # https://github.com/projectcalico/calico - release 버전확인후 조절.
      **curl -L https://github.com/projectcalico/calico/releases/download/v3.24.1/calicoctl-linux-amd64 -o calicoctl**
      **chmod +x calicoctl
      mv calicoctl /usr/bin**

      **calicoctl get ippool -o wide**
      NAME                  CIDR             NAT    IPIPMODE   VXLANMODE     DISABLED   DISABLEBGPEXPORT   SELECTOR
      default-ipv4-ippool   192.168.0.0/16   true   **Never**      CrossSubnet   false      false              all()

      **cat << END > ipipmode.yaml
      apiVersion: projectcalico.org/v3
      kind: IPPool
      metadata:
        name: default-ipv4-ippool
      spec:
        blockSize: 26
        cidr: 192.168.0.0/16
        ipipMode: Always
        natOutgoing: true
        nodeSelector: all()
        vxlanMode: Never**
      END

      **calicoctl apply -f ipipmode.yaml
      calicoctl get ippool -o wide**
      NAME                  CIDR             NAT    **IPIPMODE**   VXLANMODE   DISABLED   DISABLEBGPEXPORT   SELECTOR
      default-ipv4-ippool   192.168.0.0/16   true   **Always**     Never       false      false              all()
      ```

  - 참고: Weave CNI 설치 시 적용.
    Weavworks : [https://www.weave.works/docs/net/latest/kubernetes/kube-addon/](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) - install

    ````bash
    **kubectl get nodes**
    NAME STATUS ROLES AGE VERSION
    master NotReady control-plane 4m59s v1.24.4

            **kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml**

            **kubectl get nodes**
            NAME     STATUS   ROLES           AGE    VERSION
            master   Ready    control-plane   6m2s   v1.24.4
            ```
    ````

## 5. \***\*Joining nodes\*\***

- 노드는 워커 노드(컨테이너 및 파드 등)가 실행되는 곳입니다. 클러스터에 새 노드를 추가하려면 각 시스템에 대해 다음을 수행합니다.
  - SSH to the machine
  - Become root (e.g. `sudo -i`)
  - [Install a runtime](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime) if needed
  - Run the command that was output by `kubeadm init`
  ```bash
  **kubeadm join 172.31.X.X:6443 --token thuy4g... \
  	--discovery-token-ca-cert-hash sha256:26b18426.. \
    --cri-socket unix:///var/run/cri-dockerd.sock**
  ```

## 6. 기본 구성

- kubectl 명령어 자동 완성 설정
- [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

  ```bash
  **source <(kubectl completion bash)
  echo "source <(kubectl completion bash)" >> ~/.bashrc**
  ```

- ubuntu 사용자가 kubectl 명령 실행 가능하도록 설정

# 프로젝트 삭제

```bash
gcloud projects delete woodong-prj
```

## 참고자료

[gcloud CLI 개요](https://cloud.google.com/sdk/gcloud/)  
[gcloud 대화형 셸 사용](https://cloud.google.com/sdk/docs/interactive-gcloud?hl=ko)  
[https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)  
[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)  
[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)  
[참고: cri-docker 설치 메뉴얼](https://www.notion.so/cri-docker-48ba62ae58194ded87896ddd38849e11)  
[https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)  
[https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)
