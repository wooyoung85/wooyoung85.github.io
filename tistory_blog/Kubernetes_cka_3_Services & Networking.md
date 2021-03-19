# 1. `messaging` application 노출시키는 서비스 생성

- name: `messaging-service`
- port: `6379`
- type: `ClusterIP`

## Solution
```bash
$> kubectl expose pod messaging --type=ClusterIP --port=6379 --name=messaging-service
```

## 📖참고자료
[Using a Service to Expose Your App](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)  
[kubectl-commands (expose)](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose)

# 2. `web-dev-app` application 노출시키는 서비스 생성
 
- name: `web-dev-app-service` 
- port: `30082` 
- type: `NodePort`
- `web-dev-app` application 은 `8080` port 로 수신 중

## Solution
```bash
# pod에 달린 label 확인 (app=web-dev-app)
$> kubectl get pods --show-labels

$> kubectl create service nodeport web-dev-app-service --node-port=30082 --tcp=8080:8080 --dry-run=client -o yaml > web-dev-app-service.yaml

# selector 수정
$> vi web-dev-app-service.yaml

$> kubectl create -f web-dev-app-service.yaml
```

> `kubectl expose` 를 활용해서 service를 생성할 수도 있음

## 📖참고자료
[kubectl-commands (create service nodeport)](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-service-nodeport-em-)


# 3. Service & Pod DNS Resolution Record
> 클러스터 내에서 서비스 및 포드 이름을 조회 할 수 있는지 테스트

- Pod 생성
    - name: `nginx-resolver`
    - image: `nginx`
- Service 생성
    - name: `nginx-resolver-service`
- 결과 값은 `/root/nginx.svc` 와 `/root/nginx.pod` 에 저장

```bash
# Pod 생성
$> kubectl run nginx-resolver --image=nginx

# Service 생성
$> kubectl expose pod nginx-resolver --port=80 --target-port=80 --name=nginx-resolver-service

# Test Pod 생성
$> kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml

# dnsutils Pod에서 nslookup 명령어 실행 (nginx-resolver-service)
kubectl exec -i -t dnsutils -- nslookup nginx-resolver-service > /root/nginx.svc

# Pod  IP 정보 확인
kubectl get pods nginx-resolver -o wide

# dnsutils Pod에서 nslookup 명령어 실행 (nginx-resolver, ip: 10.244.1.11)
kubectl exec -i -t dnsutils -- nslookup 10-244-1-11.default.pod > /root/nginx.pod
```

> dns에 대한 상세 내용은 [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) 를 확인하시기 바랍니다.

## 📖참고자료
[Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)  
[DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
