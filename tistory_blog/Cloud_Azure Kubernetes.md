# Cluster 생성은 Azure Portal에서 진행

- Kubernetes 서비스 만들기
- 네트워킹 > 네트워크 구성 : `Azure CNI`
- 네트워킹 > 네트워크 정책 : `Auzre`
- 통합 > 컨테이너 레지스트리 : `sgarchiveacr`

# Prerequisite

## Azure Cli Login

```bash
SUBSCRIPTION=d56468c0-87dc-4227-acfd-d708adf1fec3
REGISTRY_NAME=sgarchiveacr

az login
az account set --subscription $SUBSCRIPTION
az acr login --name $REGISTRY_NAME
```

## Ingress Controller 생성 시 필요한 이미지 ACR에 Import

```bash
REGISTRY_NAME=sgarchiveacr
SOURCE_REGISTRY=k8s.gcr.io
CONTROLLER_IMAGE=ingress-nginx/controller
CONTROLLER_TAG=v1.0.4
PATCH_IMAGE=ingress-nginx/kube-webhook-certgen
PATCH_TAG=v1.1.1
DEFAULTBACKEND_IMAGE=defaultbackend-amd64
DEFAULTBACKEND_TAG=1.5
RES_GROUP=sgarchive
AKS_NAME=sgarchiveaks

az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$CONTROLLER_IMAGE:$CONTROLLER_TAG --image $CONTROLLER_IMAGE:$CONTROLLER_TAG
az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$PATCH_IMAGE:$PATCH_TAG --image $PATCH_IMAGE:$PATCH_TAG
az acr import --name $REGISTRY_NAME --source $SOURCE_REGISTRY/$DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG --image $DEFAULTBACKEND_IMAGE:$DEFAULTBACKEND_TAG

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

## cluster 접속

```bash
az aks get-credentials -g $RES_GROUP -n $AKS_NAME
```

# 개발환경(sgarchive-dev) 설정

```bash
REGISTRY_NAME=sgarchiveacr
ACR_URL=$REGISTRY_NAME.azurecr.io
RES_GROUP=sgarchive
AKS_NAME=sgarchiveaks
DEV_NAME=sgarchive-dev
DEV_PUB_IP_NAME=sgarchivedevpubip
DEV_DNS_LABEL=sgarchive-dev
AKS_RES_GROUP=$(az aks show --resource-group $RES_GROUP --name $AKS_NAME --query nodeResourceGroup -o tsv)
echo $AKS_RES_GROUP

# NameSpace 생성
kubectl create ns $DEV_NAME

# 공용 IP 생성
#az network public-ip create --resource-group $AKS_RES_GROUP --name $DEV_PUB_IP_NAME --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv

DEV_PUB_IP=$(az network public-ip show --resource-group $AKS_RES_GROUP --name $DEV_PUB_IP_NAME --query ipAddress -o tsv)
echo $DEV_PUB_IP

# Use Helm to deploy an NGINX ingress controller
helm install $DEV_NAME-ingress ingress-nginx/ingress-nginx \
    --version 4.0.13 \
    --namespace $DEV_NAME \
    --set controller.ingressClassResource.name=$DEV_NAME \
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
    --set controller.service.loadBalancerIP=$DEV_PUB_IP \
    --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$DEV_DNS_LABEL

# external ip 확인
kubectl --namespace $DEV_NAME get services -o wide -w $DEV_NAME-ingress-ingress-nginx-controller
kubectl --namespace $DEV_NAME get services $DEV_NAME-ingress-ingress-nginx-controller

# Public IP address of your ingress controller
DEV_ING_IP=$(kubectl --namespace $DEV_NAME get services $DEV_NAME-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $DEV_ING_IP

# FQDN
az network public-ip show --resource-group $AKS_RES_GROUP --name $DEV_PUB_IP_NAME --query dnsSettings.fqdn -o tsv
```

https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli
