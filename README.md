# install-cert-manager
* 태그 : v1.5.4 기준
## 폐쇄망 구축 가이드
> 아래의 가이드는, 우선적으로 외부 네트워크 통신이 가능한 환경에서 필요한 이미지들을 tar로 다운받고, 해당 tar들을 폐쇄망으로 이동시켜 작업합니다. 

* 작업 디렉토리 생성 및 환경 변수 설정
```
mkdir -p ~/cert-manager-install
export CERT_MANAGER_WORKDIR=~/cert-manager-install
```

* 이미지 환경 변수 설정
  * 아래의 예시는 v1.5.4 기준. 버전에 맞게 
```
export CERT_MANAGER_VERSION=v1.5.4
export IMG_CERT_MANAGER_CONTROLLER=quay.io/jetstack/cert-manager-controller:$CERT_MANAGER_VERSION
export IMG_CERT_MANAGER_WEBHOOK=quay.io/jetstack/cert-manager-webhook:$CERT_MANAGER_VERSION
export IMG_CERT_MANAGER_CA_INJECTOR=quay.io/jetstack/cert-manager-cainjector:$CERT_MANAGER_VERSION
```
* 작업 디렉토리로 이동
```
cd $CERT_MANAGER_WORKDIR
```
* 외부 네트워크 통신이 가능한 환경에서 필요한 이미지 다운로드
```
sudo docker pull $IMG_CERT_MANAGER_CONTROLLER
sudo docker save $IMG_CERT_MANAGER_CONTROLLER > controller.tar

sudo docker pull $IMG_CERT_MANAGER_WEBHOOK
sudo docker save $IMG_CERT_MANAGER_WEBHOOK > webhook.tar

sudo docker pull $IMG_CERT_MANAGER_CA_INJECTOR
sudo docker save $IMG_CERT_MANAGER_CA_INJECTOR > cainjector.tar
```
* 레지스트리 환경 변수 설정
  * 아래 중괄호 안에, 폐쇄망에서 사용하는 레지스트리 IP와 Port를 기입
```
export REGISTRY={registry-ip}:{port}
```

* 생성한 이미지 tar 파일을 폐쇄망 환경으로 이동시킨 뒤 사용하려는 registry에 push.
```
sudo docker load < controller.tar
sudo docker tag $IMG_CERT_MANAGER_CONTROLLER ${REGISTRY}/jetstack/cert-manager-controller:$CERT_MANAGER_VERSION
sudo docker push ${REGISTRY}/jetstack/cert-manager-controller:$CERT_MANAGER_VERSION

sudo docker load < webhook.tar
sudo docker tag $IMG_CERT_MANAGER_WEBHOOK ${REGISTRY}/jetstack/cert-manager-webhook:$CERT_MANAGER_VERSION
sudo docker push ${REGISTRY}/jetstack/cert-manager-webhook:$CERT_MANAGER_VERSION

sudo docker load < cainjector.tar
sudo docker tag $IMG_CERT_MANAGER_CA_INJECTOR ${REGISTRY}/jetstack/cert-manager-cainjector:$CERT_MANAGER_VERSION
sudo docker push ${REGISTRY}/jetstack/cert-manager-cainjector:$CERT_MANAGER_VERSION
```

* 레지스트리에 푸시된 이미지들을 install.yaml에 반영
  * install.yaml은 install-cert-manager 레포에 루트 경로에 위치  
```
sed -i "s/quay.io/${REGISTRY}/g" install.yaml	 	 
```

* yaml 설치
```
kubectl apply -f install.yaml
```

## 외부 네트워크 통신이 가능한 환경 구축 가이드
* yaml 설치
```
kubectl apply -f install.yaml
```
