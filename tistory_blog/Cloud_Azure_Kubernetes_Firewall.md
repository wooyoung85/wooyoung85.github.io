# Prerequisite

- [Azure Cloud 사용하기](https://wooyoung85.tistory.com/75) 참고
- https://helm.sh/docs/intro/install/

# 리소스 그룹 생성

```bash
az group create --name woodong-rg --location koreacentral
```

# Azure Kubernetes

```bash
# 가상 네트워크 생성
az network vnet create \
 --resource-group woodong-rg \
 --name woodong-aks-vnet \
 --address-prefixes 10.20.0.0/16 \
 --subnet-name woodong-aks-subnet \
 --subnet-prefix 10.20.1.0/24

SUBNET_ID=$(az network vnet subnet show --resource-group woodong-rg --vnet-name woodong-aks-vnet --name woodong-aks-subnet --query id -o tsv)
echo $SUBNET_ID

# Azure Kubernetes Cluster 생성
az aks create  -y \
  --name woodong-aks \
  --resource-group woodong-rg \
  --network-plugin azure \
  --nodepool-name aksnodepool \
  --node-count 1 \
  --node-vm-size Standard_B2s \
  --vnet-subnet-id $SUBNET_ID

AKS_PRINCIPAL_ID=$(az aks show --name woodong-aks --resource-group woodong-rg --query identity.principalId -o tsv)
VNET_ID=$(az network vnet show --resource-group woodong-rg --name woodong-aks-vnet --query id -o tsv)
echo $AKS_PRINCIPAL_ID
echo $VNET_ID

az role assignment create --assignee $AKS_PRINCIPAL_ID --role "Network Contributor" --scope $VNET_ID
az role assignment list --all --assignee $AKS_PRINCIPAL_ID
```

# Azure Firewall

```bash
# 가상 네트워크 생성
az network vnet create \
  --name woodong-fw-vnet \
  --resource-group woodong-rg \
  --location koreacentral \
  --address-prefix 10.10.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.10.1.0/24

# 방화벽 생성
az network firewall create \
 --name woodong-fw \
 --resource-group woodong-rg \
 --location koreacentral

# 공용 IP 할당
az network public-ip create \
 --name woodong-pip \
 --resource-group woodong-rg \
 --location koreacentral \
 --allocation-method Static \
 --dns-name woodong \
 --sku standard

 # 방화벽 IP 구성
az network firewall ip-config create \
 --firewall-name woodong-fw \
 --name woodong-fw-config \
 --public-ip-address woodong-pip \
 --resource-group woodong-rg \
 --vnet-name woodong-fw-vnet

az network firewall update --name woodong-fw --resource-group woodong-rg

# Add Network FW Rules
az network firewall network-rule create \
  --resource-group woodong-rg \
  --firewall-name woodong-fw \
  --collection-name "aks-netrule" \
  --name "allow-all" \
  --protocols "ANY" \
  --source-addresses "*" \
  --destination-addresses "*" \
  --destination-ports "*" \
  --action allow \
  --priority 100


# Add Application FW Rules (AKS required egress endpoints)
az network firewall application-rule create \
  --resource-group woodong-rg \
  --firewall-name woodong-fw \
  --collection-name 'AKS_Global_Required' \
  --action allow \
  --priority 100 \
  --name 'required' \
  --source-addresses '*' \
  --protocols 'http=80' 'https=443' \
  --target-fqdns \
      'aksrepos.azurecr.io' \
      '*blob.core.windows.net' \
      'mcr.microsoft.com' \
      '*cdn.mscr.io' \
      '*.data.mcr.microsoft.com' \
      'management.azure.com' \
      'login.microsoftonline.com' \
      'ntp.ubuntu.com' \
      'packages.microsoft.com' \
      'acs-mirror.azureedge.net'

# Add a dnat rule to azure firewall
FW_PIP=$(az network public-ip show -g woodong-rg -n woodong-pip --query "ipAddress" -o tsv)
echo $FW_PIP

az network firewall nat-rule create \
--collection-name 'aks-natrule' \
--destination-addresses $FW_PIP \
--destination-ports 80 \
--firewall-name woodong-fw \
--name inboundrule \
--protocols Any \
--resource-group woodong-rg \
--source-addresses "*" \
--translated-port 80 \
--action Dnat \
--priority 100 \
--translated-address $ING_EXTERNAL_IP
```

---

# 가상 네트워크 연결 (AKS 🔛 Firewall)

```bash
AKS_VNET_ID=$(az network vnet list --resource-group woodong-rg --query "[?contains(name, 'woodong-aks-vnet')].id" --output tsv)
FW_VNET_ID=$(az network vnet show --resource-group woodong-rg --name woodong-fw-vnet --query id --out tsv)
echo $AKS_VNET_ID
echo $FW_VNET_ID

az network vnet peering create \
--name "aks-peer-firewall" \
--resource-group woodong-rg \
--vnet-name woodong-aks-vnet \
--remote-vnet woodong-fw-vnet \
--allow-vnet-access

az network vnet peering create \
--name "firewall-peer-aks" \
--resource-group woodong-rg \
--vnet-name woodong-fw-vnet \
--remote-vnet woodong-aks-vnet \
--allow-vnet-access
```

# AKS routing table 설정

```bash
az network route-table create --resource-group woodong-rg --name woodong-rtbl

FW_PRIVATE_IP=$(az network firewall show -g woodong-rg -n woodong-fw --query "ipConfigurations[0].privateIpAddress" -o tsv)
echo $FW_PRIVATE_IP

az network route-table route create \
  --name woodong-rt \
  --next-hop-type VirtualAppliance \
  --resource-group woodong-rg \
  --route-table-name woodong-rtbl \
  --next-hop-ip-address $FW_PRIVATE_IP \
  --address-prefix "0.0.0.0/0"


# Associate the route table to AKS
az network vnet subnet update \
  --resource-group woodong-rg \
  --vnet-name woodong-aks-vnet \
  --name woodong-aks-subnet \
  --route-table woodong-rtbl
```

# Ingress Controller 배포

```bash
az aks get-credentials -g woodong-rg -n woodong-aks

helm install nginx-ingress \
ingress-nginx/ingress-nginx \
--set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"="true" \
--set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal-subnet"="woodong-aks-subnet" \
--set controller.replicaCount=1 \
--set controller.admissionWebhooks.patch.nodeSelector."kubernetes\.io/os"=linux \
--set defaultBackend.nodeSelector."kubernetes\.io/os"=linux

$ kubectl get svc
NAME                                               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kubernetes                                         ClusterIP      10.0.0.1       <none>        443/TCP                      105m
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.0.171.200   10.20.1.33    80:31358/TCP,443:31220/TCP   47s
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.0.103.72    <none>        443/TCP                      47s

ING_EXTERNAL_IP=$(kubectl get svc nginx-ingress-ingress-nginx-controller -o custom-columns="EXTERNAL-IP:.status.loadBalancer.ingress[0].ip" | grep "\.")
echo $ING_EXTERNAL_IP
```

# Web Application 배포

```bash
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
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

> Ingress 는 아래와 같이 설정해도 됨
>
> ```yaml
> apiVersion: networking.k8s.io/v1
> kind: Ingress
> metadata:
>   name: web-ingress
> spec:
>   ingressClassName: nginx
>   rules:
>     - host: woodong.koreacentral.cloudapp.azure.com
>       http:
>         paths:
>           - pathType: Prefix
>             path: "/"
>             backend:
>               service:
>                 name: web-svc
>                 port:
>                   number: 80
> ```

```bash
kubectl apply -f ~/woodong/web-deployment.yaml
```

# TEST

```bash
FQDN=$(az network public-ip show --name woodong-pip --resource-group woodong-rg --query "dnsSettings.fqdn" --output tsv)
echo $FQDN

POD_NAME=$(kubectl get pod -l app -o custom-columns="NAME:.metadata.name" | grep web)
echo $POD_NAME

echo '<h1>Test Complete!!</h1>' > ~/woodong/index.html
kubectl exec $POD_NAME -- rm /usr/share/nginx/html/index.html
kubectl cp ~/woodong/index.html default/$POD_NAME:/usr/share/nginx/html/index.html

curl -i $FQDN
```

# Resource Group 삭제

```bash
az group delete --name woodong-rg
```

## 참고자료

[michalswi/aks-with-firewallAzure: AKS behind Azure Firewall](https://github.com/michalswi/aks-with-firewall)  
[사용자 지정 경로를 사용하여 클러스터 송신 사용자 지정](https://learn.microsoft.com/ko-kr/azure/aks/egress-outboundtype)  
[AKS(Azure Kubernetes Service)에 수신 컨트롤러 만들기](https://learn.microsoft.com/ko-kr/azure/aks/ingress-basic)  
[Integrating AKS with Azure firewall](https://tufin.medium.com/integrating-aks-cluster-with-azure-firewall-b3e56d163866)
