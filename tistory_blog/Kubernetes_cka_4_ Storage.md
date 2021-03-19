# 1. Persistent Volume 생성
- name: `pv-test`
- storage: `100Mi`
- access modes: `ReadWriteMany`
- host path: `/pv/data-test`

<br/>

## Solution
```bash
$> vi pv-test.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-test
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pv/data-test"
```

```bash
$> kubectl create -f pv-test.yaml
```

## 📖참고자료
[Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)


## 2. `emptyDir` Volume 사용하는 Pod 생성

- name: `redis-storage`
- image: redis:alpine
- volume type: `emptyDir`
- mountPath: `/data/redis`

```bash
$> kubectl run redis-storage --image=redis:alpine --dry-run=client -o yaml > redis-storage.yaml
$> vi redis-storage.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: temp-volume
  volumes:
  - name: temp-volume
    emptyDir: {}
```
```bash
$> kubectl create -f redis-storage.yaml
```

## 📖참고자료
[Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/#configure-a-volume-for-a-pod)


# 3. PV, PVC 연동 및 Pod to Use a Volume
> pv는 이미 생성되어 있고, pod manifest 파일은 주어짐

> **CheckPoint**  
> ① pv가 bound 상태로 변경되었는지 확인  
> ② pod manifest 파일의 mount path 등을 정확히 확인

- mount path: /data 
- pvc name: my-pvc


```bash
# pv는 생성되어 있음
$> kubectl get pv

# pvc manifest 파일 작성
$> vi pvc.yaml
```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
```
```bash
$> kubectl create -f pvc.yaml

# 🌟Status가 Bound로 변경됐는지 확인🌟
$> kubectl get pv

# 주어진 pod manifest 확인 및 수정
$> vi use-pv.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
      - mountPath: "/data"
        name: my-storage
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: my-pvc
```
```bash
$> kubectl create -f use-pv.yaml
```
