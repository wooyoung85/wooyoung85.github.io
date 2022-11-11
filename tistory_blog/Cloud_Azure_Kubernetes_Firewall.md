# Resource Group

```bash
az group create --name woodong-rg --location koreacentral
```

# Firewall

```bash
az network vnet create \
  --name woodong-vnet \
  --resource-group woodong-rg \
  --location koreacentral \
  --address-prefix 10.10.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.10.1.0/24

# az network vnet subnet create \
#   --name woodong-aks-subnet \
#   --resource-group woodong-rg \
#   --vnet-name woodong-vnet \
#   --address-prefixes 10.10.2.0/24

az network firewall create \
 --name woodong-fw \
 --resource-group woodong-rg \
 --location koreacentral

az network public-ip create \
 --name woodong-pip \
 --resource-group woodong-rg \
 --location koreacentral \
 --allocation-method Static \
 --sku standard

 # 시간 오래걸림
az network firewall ip-config create \
 --firewall-name woodong-fw \
 --name woodong-fw-config \
 --public-ip-address woodong-pip \
 --resource-group woodong-rg \
 --vnet-name woodong-vnet

az network firewall update \
 --name woodong-fw \
 --resource-group woodong-rg

# az network public-ip show \
#  --name woodong-pip \
#  --resource-group woodong-rg
```

---

# AKS

```bash
az network vnet create \
 --resource-group woodong-rg \
 --name woodong-aks-vnet \
 --address-prefixes 10.20.0.0/16 \
 --subnet-name woodong-aks-subnet \
 --subnet-prefix 10.20.1.0/24

SUBNET_ID=$(az network vnet subnet show --resource-group woodong-rg --vnet-name woodong-aks-vnet --name woodong-aks-subnet --query id -o tsv)
echo $SUBNET_ID

az aks create \
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

# Network

## Create UDR (User Defined Route) and add a route for Azure Firewall

https://docs.microsoft.com/en-us/azure/aks/egress-outboundtype#create-a-udr-with-a-hop-to-azure-firewall

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

```

## Add Network FW Rules

https://docs.microsoft.com/en-us/azure/aks/egress-outboundtype#adding-network-firewall-rules

```bash
az network firewall network-rule create \
  --resource-group woodong-rg \
  --firewall-name woodong-fw \
  --collection-name "aksfwcollection" \
  --name "allow-all" \
  --protocols "ANY" \
  --source-addresses "*" \
  --destination-addresses "*" \
  --destination-ports "*" \
  --action allow \
  --priority 100
```

## Add Application FW Rules (AKS required egress endpoints)

https://docs.microsoft.com/en-us/azure/aks/egress

```bash
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
```

# Associate the route table to AKS

https://docs.microsoft.com/en-us/azure/aks/egress-outboundtype#associate-the-route-table-to-aks

```bash

az network vnet subnet update \
  --resource-group woodong-rg \
  --vnet-name woodong-aks-vnet \
  --name woodong-aks-subnet \
  --route-table woodong-rtbl
```

# Peering two VNets (AKS + Firewall)

```bash
AKS_VNET_ID=$(az network vnet list --resource-group woodong-rg --query "[?contains(name, 'woodong-aks-vnet')].id" --output tsv)
FW_VNET_ID=$(az network vnet show --resource-group woodong-rg --name woodong-vnet --query id --out tsv)
echo $AKS_VNET_ID
echo $FW_VNET_ID

az network vnet peering create \
--name "aks-peer-firewall" \
--resource-group woodong-rg \
--vnet-name woodong-aks-vnet \
--remote-vnet woodong-vnet \
--allow-vnet-access

az network vnet peering create \
--name "firewall-peer-k8s" \
--resource-group woodong-rg \
--vnet-name woodong-vnet \
--remote-vnet woodong-aks-vnet \
--allow-vnet-access
```

# Deploy ingress controller

https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip#create-an-ingress-controller

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
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.0.0.1 <none> 443/TCP 19m
nginx-ingress-ingress-nginx-controller LoadBalancer 10.0.151.83 10.20.2.35 80:30853/TCP,443:30870/TCP 88s
nginx-ingress-ingress-nginx-controller-admission ClusterIP 10.0.81.17 <none> 443/TCP 88s
```

# Run application

https://github.com/michalswi/url-shortener

```bash
kubectl apply -f yamls/urlshort.yaml
```

# Add a dnat rule to azure firewall

```bash
az network firewall nat-rule create \
--collection-name "k8s-example" \
--destination-addresses "${FWPUBLICIP}" \
--destination-ports 80 \
--firewall-name "${FWNAME}" \
--name inboundrule \
--protocols Any \
--resource-group "${FWRG}" \
--source-addresses "\*" \
--translated-port 80 \
--action Dnat \
--priority 100 \
--translated-address "10.20.2.35" << ingress controller external ip
```

# FQDN firewall's public IP

```bash
$ az network public-ip show \
--name ${FWPUBLICIPNAME} \
--resource-group "${FWRG}" \
--query "dnsSettings.fqdn" --output tsv

msus.westeurope.cloudapp.azure.com << 'dns_prefix' in variable.tf

curl -i msus.westeurope.cloudapp.azure.com/us/home
curl -sS msus.westeurope.cloudapp.azure.com/us/health | jq
```

```bash
az group delete --name woodong-rg

```
