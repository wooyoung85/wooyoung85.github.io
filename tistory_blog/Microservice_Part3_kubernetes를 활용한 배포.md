> 마이크로 서비스 실습을 위해 시리즈로 Posting할 예정입니다. 
>  
> freeCodeCamp의 [Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882) 블로그 글을 토대로 작성하였습니다.
>
> 제 코드는 [여기서](https://github.com/wooyoung85/kuber-sample-apps/tree/3.Orchestrating_With_Kubernetes) 확인하실 수 있습니다.

# Kubernetes란?
## 1. 개념
> container orchestration tool

### kubernetes가 하는 일
- 여러 host (= node in k8s) 를 묶어 클러스터를 구성
- container를 적절한 위치에 배포 (auto-placement)
- container 가 죽으면 자동으로 복구 (auto-restart)
- 필요에 따라 container 를 매끄럽게 추가(scaling), 복제(replication), 업데이트(rolling update), 롤백(rollback)

### 구성요소
- cluster  
  - 하나의 클러스터 안에  Master와 여러개의 Node로 구성
- master  
  - 노드를 관리하는 주체
- node  
  - VM 또는 실제 머신을 의미
- pod  
  - Containerize app이 배포되는 컴포넌트 (가장 작은 단위)  
  - Pod는 동일한 실행 환경을 공유하고 하나 이상의 컨테이너로 구성 될 수 있음
- deployment  
  - 애플리케이션의 배포/삭제, Scale out의 역할
- service  
  - Pod의 논리적 집합과 액세스 정책을 정의하는 추상화 개념

🙉 Kubernetes 관련 자세한 내용은 따로 찾아보시는게 좋을 것 같습니다 ^^

## 2. Minikube 로 Kubernetes 실습 환경 구축하기
### Minikube
👉 Minikube를 설치하면 Virtual Box에 가상머신이 하나 생기고 작은 Kubernetes 클러스터가 설정됨(Kubernetes를 Local PC환경에서 사용해 볼 수 있음)

- Virtual Machine 설치 (이미 설치가 되어 있다면 Pass)
    ```shell
    $ brew cask install virtualbox
    ```
- Minikube 설치

    ```shell
    $ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.30.0/minikube-darwin-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
    
    # brew 명령어를 통해서도 설치 가능
    $ brew cask install minikube
    ```
- Minikube 시작  

    ```shell
    minikube start
    minikube status
    ```

### kubectl
👉 Kubernetes 클러스터에 연결하는 데 사용하는 클라이언트 Command Line

- kubectl 설치
    ```shell
    curl -Lo kubectl http://storage.googleapis.com/kubernetes-release/release/v1.11.4/bin/darwin/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin/
    
    # brew 명령어를 통해서도 설치 가능
    brew install kubernetes-cli
    ```
- kubectl 확인
    ```shell
    kubectl status
    ```


😎 Docker for Mac edge 버전에서 Kubernetes를 지원하기 때문에 아래 글을 참고해서 환경을 구축하는 것도 가능할 것으로 보입니다.  
[Docker for Mac으로 Kubernetes 로컬에서 사용하기](https://blog.outsider.ne.kr/1346) 



# Kubernetes를 활용하여 Application 배포하기

## kubernetes 관련 폴더 및 파일 만들기
```
# 프로젝트 구조
kuber-sample-apps
└─── django_app
└─── kubernetes_deploy
    └─── celery
    └─── django
    └─── react
    └─── redis
    └─── spring_boot
└─── react_app
└─── springboot_app    
```

😎 `kubernetes_deploy` 폴더 안에 들어갈 `deployment.yaml`과 `service.yaml` 파일들은 [GitHub](https://github.com/wooyoung85/kuber-sample-apps/tree/3.Orchestrating_With_Kubernetes/kubernetes_deploy)를 참고하시기 바랍니다.

## Kubernetes Cluster 리소스 정의
Kubernetes Cluster 리소스는 크게 4가지로 구성되어 있음
- apiVersion : 사용하는 스키마 버전
- kind : 만들려는 리소스의 타입 정의
- metadata : 리소스 이름 부여
- spec : 생성하려는 리소스의 종류나 타입에 따라 다양함

### Deployment.yaml 정의
```yaml
apiVersion: extensions/v1beta1
kind: Deployment                                # 1
metadata:
  name: react-app
spec:
  replicas: 2                                   # 2
  minReadySeconds: 15
  strategy:
    type: RollingUpdate                         # 3
    rollingUpdate: 
      maxUnavailable: 1                         # 4
      maxSurge: 1                               # 5
  template:                                     # 6
    metadata:
      labels:
        app: react-app                          # 7
    spec:
      containers:
        - image: wooyoung85/react_app
          imagePullPolicy: Always               # 8
          name: react-app
          ports:
            - containerPort: 80
```
1. `kind` : Deplyment를 정의하는 파일이니까 Deployment
2. `spec.replicas` : 배포 시 얼마나 많은 Pod를 실행할 것인지 정의
3. `spec.strategy.type` : 현재 버전에서 다음 버전으로 업데이트 할 때 사용되는 전략을 지정하고, RollingUpdate 는 가동 중단 시간 제로를 보장함
4. `spec.strategy.rollingUpdate.maxUnavailable` : 롤링 업데이트를 수행 할 때 허용되는 최대 사용불가 Pod를 지정.  
ex) 1로 지정할 경우 하나씩 Pod를 종료시킨 후 업데이트를 수행하기 때문에 서비스는 사용가능한 상태가 계속 유지될 수 있음
5. `spec.strategy.rollingUpdate.maxSurge` : 롤링 업데이트 시 추가되는 Pod의 최대 크기를 정의.
6. `spec.template` : 새 Pod를 생성하는 데 사용할 포드 템플릿을 지정
7. `spec.template.metadata.labels.app` : Pod에 이름표를 붙여 service.yaml 파일로 Pod의 논리적 집합과 액세스 정책을 정의할 때 활용함
8. `spec.template.spec.containers.imagePullPolicy` : Always로 설정하면 컨테이너를 배치할 때마다 Docker 이미지를 가져옴

### Deployment 실행
```shell
$ kubectl apply -f deployment.yaml
```

### Service.yaml 정의

```yaml
apiVersion: v1
kind: Service              # 1
metadata:
  name: react-app-lb
spec:
  type: LoadBalancer       # 2
  ports:
  - port: 80               # 3
    protocol: TCP          # 4
    targetPort: 80         # 5
  selector:                # 6
    app: react-app         # 7
```
1. `kind` : Service를 정의하는 파일이니까 Service
2. `spec.type` : Pod들의 부하를 분산하기 위해 LoadBalancer 선택
3. `spec.ports.port` : 서비스가 요청받는 Port 지정
4. `spec.ports.port.protocol` : 통신 방법 지정
5. `spec.ports.port.targetPort` : 요청을 전달하는 Port 번호
6. `spec.selector`: Pod를 선택하기 위한 속성들을 포함하고 있음
7. `spec.selector.app`: target이 될 Pod를 정의  
ex) "react-app"이라고 정의하면 "react-app" 이름을 가진 Pod들만 Service가 적용됨

### Service 실행
```shell
$ kubectl apply -f service.yaml
```
### 실습 명령 모음
```shell
# Minikube 시작
$ minikube start
$ minikube status

# 폴더 이동
$ cd  kubernetes_deploy

# yaml파일 별로 실행해도 되지만 폴더 단위로 실행하는 것도 가능함

# Python Application
$ kubectl apply -f celery/
$ kubectl apply -f redis/
$ kubectl apply -f django/

# Spring Boot Application
$ kubectl apply -f srping_boot/

# React Application
$ kubectl apply -f react/

# 서비스 확인
$ minikube service react-app-lb
```

### 🙉 실습 시 유의사항
1. Minikube의 경우 External-IP는 `pending` 상태가 정상임  
GCP나 AWS같은 상용 클라우드 환경을 사용하게 되면 Public-IP가 할당되고, 외부에서도 접근이 가능하게 됨
    ```shell
    $ kubectl get svc react-app-lb
    NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    react-app-lb   LoadBalancer   10.100.118.99   <pending>     80:30440/TCP   23d
    ```

2. URL 환경 변수 Kube-DNS로 변경  
  - Redis 설정
    ```yaml
    ...
    containers:
        - name: django-app
          image: wooyoung85/django_app
          ports:
            - containerPort: 8000
          env:
            - name: REDIS_HOST
              value: redis-service
    ```
  - Spring Boot Application에 설정된 Djagno Application URL을 아래와 같이 Kube-DNS로 변경
    ```yaml
    ...

    containers:
        - image: wooyoung85/springboot_app
          imagePullPolicy: Always
          name: spring-boot-app
          env:
            - name: DJANGO_APP_URL
              value: "http://django-service"
          ports:
            - containerPort: 8080
    ```

3. `react-app`은 deployment를 apply하기 전에 따로 설정이 필요함!!!  
React Application(Client)에서 Spring Boot API 로 요청을 할 때 서버 환경 변수 같은 설정을 사용할 수 없기 때문에 수동으로 spring-boot-lb 서비스에 대한 url을 알아내서 넣어줘야 함

  - minikube service 리스트 보기 
    ```shell
    $ minikube service list
    |-------------|----------------------|-----------------------------|
    |  NAMESPACE  |         NAME         |             URL             |
    |-------------|----------------------|-----------------------------|
    | default     | django-service       | No node port                |
    | default     | kubernetes           | No node port                |
    | default     | react-app-lb         | http://192.168.99.100:30440 |
    | default     | redis-service        | No node port                |
    | default     | spring-boot-lb       | http://192.168.99.100:31525 |
    | kube-system | kube-dns             | No node port                |
    | kube-system | kubernetes-dashboard | http://192.168.99.100:30000 |
    |-------------|----------------------|-----------------------------|
    ```
  - `react_app/src/App.js` 파일 수정
    ```javascript
      analyzeSentence() {
          // spring-boot-lb 서비스에 대한 url을 넣어줘야 함
          fetch('http://192.168.99.100:31525/sentiment', {
              method: 'POST',
              headers: {
                  'Content-Type': 'application/json'
              },
              body: JSON.stringify({sentence: this.textField.getValue()})
          })
              .then(response => response.json())
              .then(data => this.setState(data));
      }
    ```

## 참고자료
[[Kubernetes 활용(1/8)] 시작하기](http://tech.cloudz-labs.io/posts/kubernetes/getting-start/)  
[5 minute local Kubernetes Cluster on Mac](https://gist.github.com/jloveland/df1bdec4705220eb5990)  
[Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)  
[Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-minikube/)
