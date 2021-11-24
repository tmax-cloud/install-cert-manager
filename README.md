# install-cert-manager (준비중)

## Prerequisites
* Kubernetes
## Installation (vX.Y.Z 버전 유의)
```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/vX.Y.Z/cert-manager.yaml
```
## Uninstallation (vX.Y.Z 버전 유의)
1. 모든 cert-manager 리소스들 삭제. (아래 커맨드를 이용해서 우선 어떤 cert-manager 리소스들 리스팅하고 삭제)
```bash
kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges --all-namespaces
```
2. cert-manager 삭제
```bash
kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/vX.Y.Z/cert-manager.yaml
```  
