```bash
$

az vm image list --offer CentOS --all
az group create --name openSslTest --location koreacentral
az vm create --resource-group openSslTest --name openSslTestVM --image OpenLogic:CentOS:7.5:latest --admin-username wyseo --admin-password 'qwer1234!@#$'
ssh wyseo@$(az vm show -d -g openSslTest -n openSslTestVM --query publicIps -o tsv)
az group delete --name openSslTest
```
