# 1. kubernetes 버전 업그레이드 (1.17 👉1.18.0)

- `kubeadm` 사용
- Master Node와 Worker Node가 각각 1개씩 있음 (Node 수는 총 2개)
- 업그레이드는 마스터 노드부터 시작하여 한 번에 한 노드 씩 수행되어야 함
- 다운타임을 최소화하기 위해, 기존에 배포되어 있는 `deployment`(name : `dev-nginx`)는 각 노드가 업그레이드 되기 전에 다른 노드로 reschedule 되어야 함
- 최종적으로 pod 들은 Master Node에 떠 있어야 함

> **kubernetes 버전 업그레이드는 1단계 씩만 가능함**  
1.16 👉1.18 : 불가  (1.17로 먼저 업그레이드 해야만 함)  
1.17 👉1.18 : 가능


## Solution

### Master Node 작업

```bash
$> kubeadm version

#Cluster Version 확인 필요
$> kubeadm upgrade plan
[upgrade/versions] Cluster version: v1.17.0
[upgrade/versions] kubeadm version: v1.17.0

$> apt-get install kubeadm=1.18.0-00

# 1.18.0으로 변경된 것 확인
$> kubeadm version

# Master 노드에서 모든 pod 제거
$> kubectl drain master --ignore-daemonsets --delete-local-data --force
$> kubeadm upgrade plan v1.18.0

# Kubernetes Version Upgrade 
$> kubeadm upgrade apply v1.18.0

# master node에 새 Pod가 배치가능하도록 (Mark node as schedulable)
$> kubectl uncordon master

# kubelet, kubectl 설치
$> apt-get install kubelet=1.18.0-00 kubectl=1.18.0-00
$> systemctl daemon-reload
$> systemctl restart kubelet
```

### Worker Node 작업

```bash
# node01 노드에서 모든 pod 제거
$> kubectl drain node01 --ignore-daemonsets --delete-local-data --force

# node1 접속
$> ssh node01

$> apt-get install kubeadm=1.18.0-00

# Node Upgrade
$> kubeadm upgrade node

# kubelet, kubectl 설치
$> apt-get install kubelet=1.18.0-00 kubectl=1.18.0-00
$> systemctl daemon-reload
$> systemctl restart kubelet

# node1 빠져나오기
$> logout

# node01에 새 Pod가 배치가능하도록 (Mark node as schedulable)
$> kubectl uncordon node01
```

### 최종확인
```bash
# make sure this is scheduled on master node
$> kubectl get pods -o wide | grep dev
```

## 📖참고자료
[Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)  
[Safely Drain a Node while Respecting the PodDisruptionBudget](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)


# 2. ETCD 백업

- backup location: master node `/opt/etcd-backup.db`


## Solution
```bash
$> export ETCDCTL_API=3 

# etcdctl 명령어 확인
$> etcdctl member list --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379

# etcd 백업
$> etcdctl snapshot save /opt/etcd-backup.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379

# etcd 백업 확인
$> etcdctl snapshot status /opt/etcd-backup.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379
```

> 문제에서 `--cacert`, `--cert`, `--key` 파일 경로가 안 주어진다면 `/etc/kubernetes/manifests/etcd.yaml` 파일에서 확인

## 📖참고자료
[Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)




## 6. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace.

The private key exists in the location: /root/john.key and csr at /root/john.csr

- CSR: john-developer Status:Approved
- Role Name: developer, namespace: development, Resource: Pods
- Access: User 'john' has appropriate permissions

```bash
$> cat john.csr | base64 | tr -d "\n"
$> vi john-csr.yaml
```
```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata: 
  name: john-developer
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt0L2t4QjlFb2h5SFcrK0hiVEMvSUdUMWRsbW9UOGZhUmhDc3VqNFRTNU9oWFNjCm53VzAxVVdPSm9LMUhnQStHNFVVV1B1VWNrS2VnblZXRFo3Zkxoa0xST3pLdkFDdEpTazNrMFVyQnVhUlFhK2YKV2ROMnAyV3NqamkrU2FTNWYrUnl3VTRnOHBLYVFCeXU0OXNZcWhlRU1WK2NzYkhpbWhBOTRKM0JSeld1NzBuNAphZFFpMHhOV0NmbnZuaThzU2dYWERYQ25GeFZ6SjZtNzhhSmxxeWV4TzUxSkFMSjIwMHp2WE5GeWx3VTlReE9WClY4am5tQi9LbW9ERGxTR1ZUV0pZY2RRVVBVN0xRZnUxb2YyL1J6MHd2cUp0R2VkVndLWDAybHhsQ2N0WnFuRFgKNWYvT2g4WEF3eVdKY3V6MkIxbFNwNFB1YjlFZ3doYTI4cWJ6RjZFQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQkVTcHp5YWxWcWNzN29zVmxWalNObXAwY3EwNDdQaG1JMkw0d0JrMklqY3FURGdsUFVwT0JmCko0Vi9iUURBSnVlTndrLzNLTnRGY1RHdEpzcS9HOTBWQzFWVjA1VjVRSys5bUo1RE1TcE5xRUkxUi9rei9NVlEKQktHSHRuZVFCRExKVGozL3R5QUVTV084QTNOOUdxSVhnRm02OUcrTGdIQnJIUGZUK21qdndodFFhZjcrbTFNNgorL3lIVVg5bnJ3aFVpL2VXSlNEZzJMNzgzNUhSNzJWTktuZkM0alRJTWtLQWxveklNd1pRY2tvQVo0VFBtRmErCkZ0c1YxYUszUkxIOUJvY01UUkRubTd1M1hkMnhZeHF1a0pWY0FVRHBSOHhzMXh6NTNzNjkyRHprMlE0U244Z2sKNnRkRnpEWkRteFUzaHF2cUpHMEFKODB0NmVGcU84VzMKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - server auth
```
```bash
$> kubectl create -f john-csr.yaml
$> kubectl get csr
$> kubectl certificate approve john-developer
$> kubectl get csr
$> kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods --namespace=development
$> kubectl create rolebinding developer-binding-john --role=developer --user=john --namespace=development
# 확인
$> kubectl auth can-i update pods --namespace=development --as=john
```



## 8. Create a static pod on node01 called nginx-critical with image nginx. Create this pod on node01 and make sure that it is recreated/restarted automatically in case of a failure.
Use /etc/kubernetes/manifests as the Static Pod path for example.

- Kubelet Configured for Static Pods
- Pod nginx-critical-node01 is Up and running

```bash
master $> kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > nginx-critical.yaml
master $> ssh node01
node01 $> cat /var/lib/kubelet/config.yaml | grep -i staticPodPath
staticPodPath: /etc/kubernetes/manifests

node01 $> mkdir -p /etc/kubernetes/manifests
node01 $> cd /etc/kubernetes/manifests/
node01 $> vi nginx-critical.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-critical
  labels:
    run: nginx-critical
spec:
  containers:
  - image: nginx
    name: nginx-critical
```

### 확인 명령어
```bash
node01 $> docker ps | grep nginx
master $> kubectl get pods --all-namespaces
```
