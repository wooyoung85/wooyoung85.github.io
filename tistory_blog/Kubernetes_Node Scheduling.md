# Node Scheduling

## Node 선택
> 접근방식 : **Labels and Selectors**  

🌟Node 에 라벨이 추가되어 있어야 NodeName, NodeSelector에 의한 Node 선택이 가능함
```bash
$> kubectl label nodes <노드 이름> <레이블 키>=<레이블 값>
```

### NodeName
- 간단한 형태의 노트 선택 방법이지만 일반적으로는 사용하지 않음  
(NodeName은 변할 수 있기 때문에)
- 스케줄러에 의해 pod가 배치되는 것이 아니라 해당 노드에서 kubelet 실행을 통해 파드를 실행함

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: kube-01
```

### NodeSelector
- key-value 쌍이 정확하게 일치해야 함  
(정확히 매치가 안되면 어느 노드에도 배치가 안되서 에러남)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

### NodeAffinity
- NodeSelector 의 단점 보완	👉 더 많은 연산자 제공
- `requiredDuringSchedulingIgnoredDuringExecution` : 파드가 규칙을 만족하는 노드에 스케줄 되도록 함 (nodeSelector 와 같으나 보다 표현적인 구문을 사용함)
- `preferredDuringSchedulingIgnoredDuringExecution` :  스케줄러가 시도하려고는 하지만, 보증하지 않음

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

## Pod 간 집중/분산
> 접근방식 : Labels and Selectors

- 세 개의 노드가 있는 클러스터에서 웹 서버와 캐시가 함께 위치하도록 yaml 파일 작성

    |node1|node2|node3|
    |-|-|-|
    |webserver-1|webserver-2|webserver-3|
    |cache-1|cache-2|cache-3|

- redis-server(cache) 분산 배치되도록 `podAntiAffinity` 설정
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: redis-cache
    spec:
    selector:
        matchLabels:
        app: store
    replicas: 3
    template:
        metadata:
        labels:
            app: store
        spec:
        affinity:
            podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                    operator: In
                    values:
                    - store
                topologyKey: "kubernetes.io/hostname"
        containers:
        - name: redis-server
            image: redis:3.2-alpine
    ```

- web-app 이 분산 배치 배치 되면서 redis-server 와 함께 위치하게끔 하기 위해 `podAntiAffinity`, `podAffinity` 설정
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: web-server
    spec:
    selector:
        matchLabels:
        app: web-store
    replicas: 3
    template:
        metadata:
        labels:
            app: web-store
        spec:
        affinity:
            podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                    operator: In
                    values:
                    - web-store
                topologyKey: "kubernetes.io/hostname"
            podAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                    operator: In
                    values:
                    - store
                topologyKey: "kubernetes.io/hostname"
        containers:
        - name: web-app
            image: nginx:1.16-alpine
    ```

## Node에 할당제한

### Toleration / Taint
> 노드에 Taint 설정을 하게 되면 Toleration 설정 없이는 그 노드에 Pod가 할당될 수 없음


- 노드에 Taint 추가
    ```bash
    $> kubectl taint nodes node1 key1=value1:NoSchedule
    ```

- Pod에 Toleration 설정
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
    ```