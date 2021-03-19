# 1. `kubeconfig` 파일 Trouble Shooting
- kubeconfig 파일은 `/root/admin.kubeconfig` 에 있음

```bash
# 마스터 및 서비스의 주소 표시
$> kubectl cluster-info

# kubeconfig 파일 확인
# port 번호가 잘못되어 있음을 확인 !!
$> cat admin.kubeconfig

# kubeconfig 파일 수정
$> sed -i "s/:2379/:6443/g" admin.kubeconfig
```

## 📖참고자료
[Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)



# 2. mysql Pod Trouble Shooting

- `dev` namespace에 `dev-mysql` deployment 가 배포되어 있음
- `dev-mysql` deployment 상세 spec
    - `dev-pv` volume 을 `/var/lib/mysql` 경로로 마운트하고 있음
    - `MYSQL_ALLOW_EMPTY_PASSWORD=1` 환경변수를 사용하고 있음

> 주의사항: persistent volume은 수정하거나 삭제하면 안됨

```bash
# Pod 상세내역 중 Eventes 확인
# pod 생성 시 pvc가 정의되었으나 실제로 생성되어 있지 않음
$> kubectl -n dev describe pod dev-mysql-57d78bdbc6-9zpnj
...
Events:
  Type     Reason            Age        From               Message
  ----     ------            ----       ----               -------
  Warning  FailedScheduling  <unknown>  default-scheduler  persistentvolumeclaim "mysql-dev-pvc" not found
  Warning  FailedScheduling  <unknown>  default-scheduler  persistentvolumeclaim "mysql-dev-pvc" not found

$> vi mysql-dev-pvc.yaml
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
  name: mysql-dev-pvc
  namespace: dev
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
```
```
$> kubectl create -f mysql-dev-pvc.yaml
```

## 📖참고자료
[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)


# 3. Pod Trouble Shooting
- pod 가 하나 배포되어 있음 (name: `red`)

## Solution
```bash
# 상태 확인
$> kubectl describe pods red

# 로그 확인
$> kubectl logs red
Error from server (BadRequest): container "red-container" in pod "red" is waiting to start: PodInitializing

$> kubectl get pods red -o yaml > red.yaml

# initContainer command 가 오류(오타)가 있어서 에러남
# initContainer command 수정
$> vi red.yaml

# pod 삭제 후 재 생성
$> kubectl delete pod red
$> kubectl create -f red.yaml
```

## 📖참고자료
[Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/#debugging-pods)