### 자동완성

```bash
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc

source ~/.bashrc
```

### Kubernetes Cluster 정보 확인

```bash
k config current-context
k config use-context hk8s
# control-plane, worker node 이름 추출
k get nodes | cut -d' ' -f1 | grep -v NAME
```

### Multi Cluster 정보 확인

```bash
k config current-context
k config use-context k8s
# Cluster 상태 확인
k cluster-info

# CNI 확인
ssh k8s-master
user@k8s-master> ll /etc/cni/net.d/
10-flannel.conflist

# Ready 상태인 노드 이름 추출
k get nodes | grep -iw ready | cut -d' ' -f1
```

### Etcd Backup & Restore

```bash
ssh k8s-master

#trusted-ca-file 경로 (/etc/kubernetes/pki/etcd/ca.crt)
user@k8s-master> ps -ef | grep kube | grep -- --trusted-ca-file
root       2377   2310  1 07:48 ?        00:00:43 etcd --advertise-client-urls=https://10.0.2.10:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --initial-advertise-peer-urls=https://10.0.2.10:2380 --initial-cluster=k8s-master=https://10.0.2.10:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://10.0.2.10:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://10.0.2.10:2380 --name=k8s-master --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

# cert-file 경로 (/etc/kubernetes/pki/etcd/server.crt)
user@k8s-master> ps -ef | grep kube | grep -- --cert-file

# ker-file 경로 (/etc/kubernetes/pki/etcd/server.key)
user@k8s-master> ps -ef | grep kube | grep -- --key-file

# Backup
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/etcd-backup
```
