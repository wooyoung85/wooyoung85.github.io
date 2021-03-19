# 1. 아래와 같은 양식으로 `test01` 네임스페이스에 있는 모든 deployments 출력

|DEPLOYMENT|CONTAINER_IMAGE|READY_REPLICAS|NAMESPACE|
|-|-|-|-|
|`<deployment name>`|`<container image used>`|`<ready replica count>`|`<Namespace>`|

- deployment 이름의 오름차순으로 정렬
- 결과 값은 `/opt/test01_data` 에 저장


## Solution
```bash
$> kubectl -n test01 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/test01_data
```

## 📖참고자료
[Overview of kubectl](https://kubernetes.io/docs/reference/kubectl/overview/#custom-columns)


# 2. deployment 생성 후 rolling update 하기

- 새 deployment 생성 
    - name: `nginx-deploy`
    - image: `nginx:1.16`
    - replica: `1`
- 1.17 버전으로 이미지 rolling update 
- version upgrade 가 기록되어야 함


## Solution
```bash
$> kubectl create deployment nginx-deploy --image=nginx:1.16
$> kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
$> kubectl rollout history deployment/nginx-deploy
```

## 📖참고자료
[Performing a Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)  
[kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

# 3. Pod에서 Secret을 File로 사용하기

- 다음 조건으로 Pod 생성하기
    - name: `secret-pod`
    - namespace: `secret-ns`
    - image: `busybox`

- Pod 안에 들어갈 Container 조건
    - name: `secret-con`
    - command: `sleep 4800`
    - secret volume 마운트 
        - read-only 👌
        - mount path: `/etc/secret-volume`

> `dotfile-secret` 이라는 이름으로 secret 은 이미 만들어져 있음


## Solution
```bash
$> kubectl -n secret-ns run secret-pod --image=busybox --dry-run=client -o yaml > secret-pod.yaml

$> vi secret-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret-ns
spec:
  containers:
  - image: busybox
    name: secret-con
    command: ["sleep", "4800"]
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
```

```bash
$> kubectl create -f secret-pod.yaml
```

## 📖참고자료
[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)  


# 3. Deploy a pod named nginx-pod using the nginx:alpine image.
- Name: nginx-pod
- Image: nginx:alpine

```bash
$> kubectl run nginx-pod --image=nginx:alpine
```

# 4. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
- Pod Name: messaging
- Image: redis:alpine
- Labels: tier=msg

```bash
$> kubectl run messaging --image=redis:alpine --labels="tier=msg"
```

# 5. Create a namespace named apx-x9984574
- Namespace: apx-x9984574
```bash
$> kubectl create ns apx-x9984574
```


# 6. Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json
```bash
$> kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
```


## 6. Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas
- Name: hr-web-app
- Image: kodekloud/webapp-color
- Replicas: 2

```bash
$> kubectl create deployment hr-web-app --image=kodekloud/webapp-color
$> kubectl scale deployment hr-web-app --replicas=2
```


## 7. Create a static pod named static-busybox on the master node that uses the busybox image and the command sleep 1000
- Name: static-busybox
- Image: busybox

```bash
$> kubectl run static-busybox --image=busybox --dry-run=client -o yaml > static-busybox.yaml
$> vi static-busybox.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - image: busybox
    name: static-busybox
    command: ["sleep", "1000"]
```
```bash
$> cp static-busybox.yaml /etc/kubernetes/manifests/.
```

### Hint
```bash
$> kubectl run --restart=Never --image=busybox static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```



## 8. Create a POD in the finance namespace named temp-bus with the image redis:alpine.
- Name: temp-bus
- Image Name: redis:alpine

```bash
$> kubectl -n finance run temp-bus --image=redis:alpine
```



## 11. Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt

The osImages are under the nodeInfo section under status of each node.

```bash
$> kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
```

# 3. Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time

The container should sleep for 4800 seconds

- Pod: super-user-pod
- Container Image: busybox:1.28
- SYS_TIME capabilities for the conatiner?

```bash
$> kubectl run super-user-pod --image=busybox:1.28 --dry-run=client -o yaml > super-user-pod.yaml
$> vi super-user-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - image: busybox:1.28
    name: super-user-pod
    command: ["sleep", "4800"]
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```
```bash
$> kubectl create -f super-user-pod.yaml
```