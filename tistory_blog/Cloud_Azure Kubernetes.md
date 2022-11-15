# Prerequisite

- Azure Cli Login

  ```bash
  SUBSCRIPTION=<구독 ID>

  az upgrade
  az login
  az account set --subscription $SUBSCRIPTION
  ```

- helm Install 👉 https://helm.sh/docs/intro/install/

> [Azure Cloud 사용하기](https://wooyoung85.tistory.com/75) 참고

# 리소스 그룹 생성

```bash
RG_NAME=woodong-rg

az group create --name $RG_NAME --location koreacentral
```

# Azure Kubernetes Service

```bash
AKS_NAME=woodong-aks

az aks create \
  --name $AKS_NAME \
  --resource-group $RG_NAME \
  --network-plugin azure \
  --nodepool-name aksnodepool \
  --node-count 1 \
  --node-vm-size Standard_B2s
```

# Azure Container Registry

```bash
ACR_NAME=woodongacr

# Azure Container Registry 생성
az acr create --resource-group $RG_NAME --name $ACR_NAME --sku Basic
# AKS에 ACR Attach
az aks update -g $RG_NAME -n $AKS_NAME --attach-acr $ACR_NAME
# Azure Container Registry 로그인
az acr login --name $ACR_NAME
```

# Ingress Controller

```bash
SOURCE_REGISTRY=registry.k8s.io
CONTROLLER_IMAGE=ingress-nginx/controller
CONTROLLER_TAG=v1.3.0
PATCH_IMAGE=ingress-nginx/kube-webhook-certgen
PATCH_TAG=v1.3.0
DEFAULTBACKEND_IMAGE=defaultbackend-amd64
DEFAULTBACKEND_TAG=1.5


# Ingress Controller 생성 시 필요한 이미지 ACR에 Import
az acr import --name $ACR_NAME --source $SOURCE_REGISTRY/$CONTROLLER_IMAGE:$CONTROLLER_TAG --image $CONTROLLER_IMAGE:$CONTROLLER_TAG
az acr import --name $ACR_NAME --source $SOURCE_REGISTRY/$PATCH_IMAGE:$PATCH_TAG --image $PATCH_IMAGE:$PATCH_TAG
az acr import --name $ACR_NAME --source $SOURCE_REGISTRY/$DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG --image $DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG

# Add the ingress-nginx repository
# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

```bash
# cluster 접속
az aks get-credentials -g $RG_NAME -n $AKS_NAME

ACR_URL=$ACR_NAME.azurecr.io
PUB_IP_NAME=woodong-pip
DNS_LABEL=woodong
AKS_RG_NAME=$(az aks show --resource-group $RG_NAME --name $AKS_NAME --query nodeResourceGroup -o tsv)
echo $AKS_RG_NAME

# 공용 IP 생성
az network public-ip create --resource-group $AKS_RG_NAME --name $PUB_IP_NAME --sku Standard --allocation-method static

PUB_IP=$(az network public-ip show --resource-group $AKS_RG_NAME --name $PUB_IP_NAME --query ipAddress -o tsv)
echo $PUB_IP

# NGINX Ingress Controller 배포
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --version 4.0.13 \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.image.registry=$ACR_URL \
    --set controller.image.image=$CONTROLLER_IMAGE \
    --set controller.image.tag=$CONTROLLER_TAG \
    --set controller.image.digest="" \
    --set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.image.registry=$ACR_URL \
    --set controller.admissionWebhooks.patch.image.image=$PATCH_IMAGE \
    --set controller.admissionWebhooks.patch.image.tag=$PATCH_TAG \
    --set controller.admissionWebhooks.patch.image.digest="" \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.image.registry=$ACR_URL \
    --set defaultBackend.image.image=$DEFAULTBACKEND_IMAGE \
    --set defaultBackend.image.tag=$DEFAULTBACKEND_TAG \
    --set defaultBackend.image.digest="" \
    --set controller.service.externalTrafficPolicy=Local \
    --set controller.service.loadBalancerIP=$PUB_IP \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$DNS_LABEL
```

> kubernetes-announce 그룹에서 공지한 내용을 보면 Kubernetes 의 Container Registry 를 `k8s.gcr.io` 에서 `registry.k8s.io` 로 바꿨다고 하였음 (https://groups.google.com/g/kubernetes-announce/c/LQki8n9ZxCo/m/gYeQ0adTAgAJ)

> [NGINX Ingress Controller 배포를 위해 사용한 이미지 저장소]  
> `ingress-nginx/controller` : https://console.cloud.google.com/gcr/images/k8s-artifacts-prod/us/ingress-nginx/controller  
> `ingress-nginx/kube-webhook-certgen` : https://console.cloud.google.com/gcr/images/k8s-artifacts-prod/us/ingress-nginx/kube-webhook-certgen  
> `defaultbackend-amd64` : https://console.cloud.google.com/gcr/images/k8s-artifacts-prod/us/defaultbackend-amd64

# Web Application 배포

```bash
# NameSpace 생성
kubectl create ns woodong
vi ~/woodong/web-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: web
  replicas: 1
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14
---
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: woodong.koreacentral.cloudapp.azure.com
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

```bash
kubectl apply -n woodong -f ~/woodong/web-deployment.yaml
```

# TEST

```bash
FQDN=$(az network public-ip show --name $PUB_IP_NAME --resource-group $AKS_RG_NAME --query "dnsSettings.fqdn" --output tsv)
echo $FQDN

POD_NAME=$(kubectl get pod -n woodong -l app -o custom-columns="NAME:.metadata.name" | grep web)
echo $POD_NAME

echo '<h1>Test Complete!!</h1>' > ~/woodong/index.html
kubectl exec $POD_NAME -n woodong -- rm /usr/share/nginx/html/index.html
kubectl cp ~/woodong/index.html woodong/$POD_NAME:/usr/share/nginx/html/index.html

curl -i $FQDN
```

# Resource Group 삭제

```bash
az group delete --name $RG_NAME
```
