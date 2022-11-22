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

- gcloud 초기화

  ```bash
  gcloud init
  ```

- 로그인 및 프로젝트 설정

  ```bash
  gcloud auth login --no-launch-browser
  gcloud config set project <PROJECT_ID>
  gcloud config set compute/region asia-northeast3
  gcloud config set compute/zone asia-northeast3-a
  ```

  > gcloud 초기화를 했다면 로그인 및 프로젝트 설정은 따로 하지 않아도 됨

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

- 환경변수

  ```bash
  PROJECT=$(gcloud config get-value project)
  ACCOUNT=$(gcloud config get-value account)
  ZONE=$(gcloud config get-value compute/zone)
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
  rm -rf ~/.ssh
  ssh-keygen -m PEM -t rsa -b 4096 -C $ACCOUNT -f ~/.ssh/gcp_rsa
  chmod 400 ~/.ssh/gcp_rsa
  ```

- Compute Engine 에 SSH Key 등록

  ```bash
  gcloud compute config-ssh --ssh-key-file=~/.ssh/gcp_rsa
  ```

  > `~/.ssh/config` 파일에 ssh 접속 설정 정보가 셋팅됨

- SSH 접속

  ```bash
  ssh master.$ZONE.$PROJECT
  ssh worker1.$ZONE.$PROJECT
  ssh worker2.$ZONE.$PROJECT
  ```

  > 인스턴스 생성이 완료된 후 접속 시도해야 함

# Ansible

- 설치

  ```bash
  sudo apt install ansible
  ```

- Inventory 설정 파일

  ```bash
  #작업폴더
  mkdir k8s-ansible

  # Inventory
  cat <<EOF > ~/k8s-ansible/gcp.inv
  [k8s]
  master.$ZONE.$PROJECT
  worker1.$ZONE.$PROJECT
  worker2.$ZONE.$PROJECT
  [master]
  master.$ZONE.$PROJECT
  [worker]
  worker1.$ZONE.$PROJECT
  worker2.$ZONE.$PROJECT
  EOF
  ```

- ping 테스트

  ```bash
  ansible k8s -i ~/gcp.inv -m ping --private-key=~/.ssh/gcp_rsa
  ```

## Prerequisite

```bash
# 환경변수
export MASTER_IP=$(gcloud compute instances describe master --format='get(networkInterfaces[0].networkIP)')
export WORKER1_IP=$(gcloud compute instances describe worker1 --format='get(networkInterfaces[0].networkIP)')
export WORKER2_IP=$(gcloud compute instances describe worker2 --format='get(networkInterfaces[0].networkIP)')
echo $MASTER_IP
echo $WORKER1_IP
echo $WORKER2_IP

# Prerequisite Playbook
vi ~/k8s-ansible/prerequisite.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/prerequisite.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Prerequisite
  hosts: k8s
  become: true
  vars:
    sysctl_config:
      net.bridge.bridge-nf-call-iptables: 1
      net.bridge.bridge-nf-call-ip6tables: 1
      net.ipv4.ip_forward: 1
  tasks:
    - name: "host config"
      blockinfile:
        path: /etc/hosts
        state: present
        block: |
          {{ lookup('env','MASTER_IP') }} master
          {{ lookup('env','WORKER1_IP') }} worker1
          {{ lookup('env','WORKER2_IP') }} worker2
    - name: Disable SWAP
      command: swapoff -a
    - name: Disable UFW on hosts
      ufw:
        state: disabled
    - name: "kernel modules config"
      blockinfile:
        path: /etc/modules-load.d/k8s.conf
        state: present
        block: |
          overlay
          br_netfilter
        create: true
    - name: "overlay modules load"
      modprobe:
        name: overlay
        state: present
    - name: "br_netfilter modules load"
      modprobe:
        name: br_netfilter
        state: present
    - name: bridge taffic config
      sysctl:
        name: "{{ item.key }}"
        value: "{{ item.value }}"
        sysctl_set: yes
        state: present
        reload: yes
        ignoreerrors: yes
      with_dict: "{{ sysctl_config }}"
```

## Install Container Runtime

```bash
vi ~/k8s-ansible/install-container-runtime.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/install-container-runtime.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Install Container Runtime
  hosts: k8s
  become: true
  tasks:
    - name: Update and upgrade apt packages
      apt:
        upgrade: "yes"
        update_cache: yes
    - name: Install Docker Engine
      apt:
        name: docker.io
    - name: Start and Enable Docker Engine
      systemd:
        name: docker
        state: started
        enabled: yes
    - name: Install Cri Docker Engine
      apt:
        deb: https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.6/cri-dockerd_0.2.6.3-0.ubuntu-bionic_amd64.deb
```

## Install kubeadm

```bash
vi ~/k8s-ansible/install-kubeadm.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/install-kubeadm.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Install kubeadm
  hosts: k8s
  become: true
  tasks:
    - name: Install Package (apt-transport-https, ca-certificates, curl, net-tools, jq)
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - net-tools
          - jq
    - name: Download the Google Cloud public signing key
      get_url:
        url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
        dest: /usr/share/keyrings/kubernetes-archive-keyring.gpg
    - name: "Add the Kubernetes apt repository"
      lineinfile:
        path: /etc/apt/sources.list.d/kubernetes.list
        state: present
        line: deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://packages.cloud.google.com/apt/ kubernetes-xenial main
        create: yes
    - name: Update and upgrade apt packages
      apt:
        upgrade: "yes"
        update_cache: yes
    - name: Install Package (kubelet, kubeadm, kubectl)
      apt:
        pkg:
          - kubelet=1.24.0-00
          - kubeadm=1.24.0-00
          - kubectl=1.24.0-00
    - name: kubelet Hold Package
      dpkg_selections:
        name: kubelet
        selection: hold
    - name: kubeadm Hold Package
      dpkg_selections:
        name: kubeadm
        selection: hold
    - name: kubectl Hold Package
      dpkg_selections:
        name: kubectl
        selection: hold
```

> Kubernetes apt repository 설정 시 `https://apt.kubernetes.io/` 를 사용하면 apt update 시 404 Not Found 에러가 발생하여
> `https://packages.cloud.google.com/apt/` 로 변경하였음

## Initialize Control Plane Node

```bash
vi ~/k8s-ansible/initialize-control-plane-node.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/initialize-control-plane-node.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Initialize Control Plane Node
  hosts: master
  become: true
  tasks:
    - name: Check for etcd service port open
      shell: "netstat -plnt | grep 2379 | wc -l"
      register: etcd_running
    - name: Print
      debug: msg="{{etcd_running.stdout}}"
    - name: run kubeadm init
      command: >
        kubeadm init --pod-network-cidr=192.168.0.0/16 \
        --cri-socket unix:///var/run/cri-dockerd.sock
      when: etcd_running.stdout == '0'
    - name: Create .kube directory
      file:
        path: /root/.kube
        state: directory
    - name: copy /etc/kubernetes/admin.conf
      copy:
        src: /etc/kubernetes/admin.conf
        dest: /root/.kube/config
        owner: root
        group: root
        remote_src: yes
      when: etcd_running.stdout == '0'
- name: Install Pod network add-on
  hosts: master
  become: true
  tasks:
    - name: Check for calico-system
      shell: "kubectl get pods -n calico-system --field-selector=status.phase=Running | wc -l"
      register: calico_running
    - name: Print
      debug: msg="{{calico_running.stdout}}"
    - name: kubectl create -f tigera-operator.yaml
      command: kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml
      when: calico_running.stdout == '0'
    - name: kubectl create -f custom-resources.yaml
      command: kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml
      when: calico_running.stdout == '0'
```

## Install Calico Network Plug-In

```bash
vi ~/k8s-ansible/install-calico-network-plug-in.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/install-calico-network-plug-in.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Install Calico Network Plug-In
  hosts: master
  become: true
  tasks:
    - name: Check for Master Node Status
      shell: 'kubectl get nodes -o json | jq -r ''.items[] | select(.status.conditions[] | select(.type=="Ready")) | .metadata.name '''
      register: master_ready
      until: master_ready.stdout.find("master") != -1
      retries: 10
      delay: 10
    - name: Print
      debug: msg="{{master_ready.stdout}}"
    - name: Download calicoctl
      get_url:
        url: https://github.com/projectcalico/calico/releases/download/v3.24.1/calicoctl-linux-amd64
        dest: /root/calicoctl
    - name: Move calicoctl (1/2)
      copy:
        src: /root/calicoctl
        dest: /usr/bin/calicoctl
        owner: root
        group: root
        mode: "0755"
        remote_src: yes
    - name: Move calicoctl (2/2)
      file:
        path: /root/calicoctl
        state: absent
    - name: ipipmode.yaml
      blockinfile:
        path: /root/ipipmode.yaml
        state: present
        block: |
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
            vxlanMode: Never
        create: true
    - name: run calicoctl apply -f ipipmode.yaml
      command: calicoctl apply -f /root/ipipmode.yaml
      when: master_ready.stdout == 'master'
```

## Join Worker Node

```bash
vi ~/k8s-ansible/join-worker-node.yaml
ansible-playbook -i ~/k8s-ansible/gcp.inv ~/k8s-ansible/join-worker-node.yaml --private-key=~/.ssh/gcp_rsa
```

```yaml
- name: Get Token Info
  hosts: master
  become: true
  tasks:
    - name: get kubeadm join token
      shell: "kubeadm token list -o json | jq -r '.token'"
      register: join_token
    - name: Print
      debug: msg="{{join_token.stdout}}"
    - local_action:
        module: copy
        content: "{{join_token.stdout}}"
        dest: /tmp/join_token.out
    - name: get kubeadm join discovery-token-ca-cert-hash
      shell: "openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.\\* //' | cut -d'=' -f2 | awk '{$1=$1};1'"
      register: discovery_token_ca_cert_hash
    - name: Print
      debug: msg="{{discovery_token_ca_cert_hash.stdout}}"
    - local_action:
        module: copy
        content: "{{discovery_token_ca_cert_hash.stdout}}"
        dest: /tmp/discovery_token_ca_cert_hash.out
- hosts: worker
  tasks:
    - name: Copy each file over that matches the given pattern
      copy:
        src: "{{ item }}"
        dest: "/tmp"
      with_fileglob:
        - "/tmp/*.out"
- name: Joining Nodes
  hosts: worker
  become: true
  tasks:
    - shell: cat /tmp/join_token.out
      register: join_token
    - set_fact: f_token={{ join_token.stdout }}
    - shell: cat /tmp/discovery_token_ca_cert_hash.out
      register: discovery_token_ca_cert_hash
    - set_fact: f_discovery_token_ca_cert_hash={{ discovery_token_ca_cert_hash.stdout }}
    - name: kubeadm token
      command: >
        kubeadm join "{{ lookup('env','MASTER_IP') }}":6443 --token {{f_token}} \
        --discovery-token-ca-cert-hash sha256:{{f_discovery_token_ca_cert_hash}} \
        --cri-socket unix:///var/run/cri-dockerd.sock
```

## 참고자료

[gcloud CLI 개요](https://cloud.google.com/sdk/gcloud/)  
[gcloud 대화형 셸 사용](https://cloud.google.com/sdk/docs/interactive-gcloud?hl=ko)  
[kubeadm 설치하기](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  
[컨테이너 런타임](https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/)  
[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)  
[클러스터 네트워킹](https://kubernetes.io/ko/docs/concepts/cluster-administration/networking/)  
[Quickstart for Calico on Kubernetes](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)
