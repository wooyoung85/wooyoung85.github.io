# Enterprise MLOps 환경 구축 가이드

- 이번 글을 시작으로 Enterprise 환경에서 MLOps 환경을 구축하는 방법을 공유하고자 합니다.
- 모델 개발, 배포, 모니터링에 필요한 기술 스택과 실습 과정을 단계별로 진행할 예정입니다.
- `Kubeflow` 를 중심으로 한 오픈소스 기반의 MLOps 플랫폼 구축에 초점을 맞췄습니다.



## Prerequisite (실습환경 준비)

- 최소 요구 Spec : `CPU - 16 Core, MEM - 32 GB, DISK - 100 GB` 
- 권장 요구 Spec : `CPU - 32 Core, MEM - 64 GB, DISK - 200 GB` 
- OS : `RHEL8`, `ROCKY8` (Kernel 4.X)
- Kubeflow/Keycloak/MinIO에 적용할 Domain 
   - https://kubeflow.example.com
   - https://keycloak.example.com
   - http://minio.example.com

> 저는 Domain 발급을 위해 `AWS Route53` 서비스를 활용했습니다.  
> 실습은 WSL2 Ubuntu 에서 `gcloud cli`를 활용해 GCP Compute Engine을 생성 방식을 선호합니다.

### AWS

<a href="https://wooyoung85.tistory.com/85" target="_blank">MLOps 실습환경 구축 - AWS (with. Terraform)</a>

### GCP

<a href="https://wooyoung85.tistory.com/88" target="_blank">MLOps 실습환경 구축 - GCP</a>  
<a href="https://wooyoung85.tistory.com/87" target="_blank">MLOps 실습환경 구축 - GCP (with. Terraform)</a> 🗸

## 목차

> 🔉 내용 정리되는대로 업데이트 할 예정입니다.

### 1. Kubernetes Cluster Install
① <a href="https://wooyoung85.tistory.com/89" target="_blank">K8S Node Pre Install (Online)</a> 🗸   
② <a href="https://wooyoung85.tistory.com/90" target="_blank">K8S Install</a> 🗸   
③ <a href="https://wooyoung85.tistory.com/91" target="_blank">K8S Node Pre Install (Offline)</a>    
④ <a href="https://wooyoung85.tistory.com/92" target="_blank">K8S Node GPU Driver Install</a>    
⑤ <a href="https://wooyoung85.tistory.com/93" target="_blank">Nvidia Device Plugin Install</a>    
⑥ <a href="https://wooyoung85.tistory.com/94" target="_blank">Kubeflow Install in Kind Cluster</a>  <span style="font-size:10px;">🙋‍♂️ Kind 를 활용한 kubeflow 설치 테스트</span>

### 2. Kubeflow Install



### 3. Supporting System Install

### 4. Custom Notebook Image

### 5. Notebook Stress Test

### 6. Model Registry

### 7. KServe

### 8. Katib

### 9. Pipeline

### 10. Manifest

### 11. Admin

