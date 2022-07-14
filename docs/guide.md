# Cert-Manager
- Cert-manager는 Kubernetes 내부에서 HTTPS 통신을 위한 인증서를 생성하고, 또 인증서의 만료 기간이 되면 자동으로 인증서를 갱신해주는 역할을 하는 Certificate manager controller이다.

- k8s 내에서 외부에 존재하는 Issuer를 활용하거나, selfsigned issuer를 직접 생성하여 인증서를 생성하고, 이때 생성된 인증서를 만료기한이 가까워지면 자동으로 갱신해준다.

### ACME란?

ACME (Automatic Certificate Management Environment)는 프로토콜이다.
ACME는 PKIX( x509)를 사용하는 CA와 사용자 사이의 인증절차를 자동화 하기 위한 프로토콜이다.

(간단히 말하면 x509로 인증서를 발급받는 프로토콜이다.)

### CA란

실제 인증서를 발급해주는 발급 기관이다.

신뢰된 CA로 부터 발급 받은 인증서임을 확인할 때 필요

self-signed와 같은 자체 발급 인증서를 브라우저에서 신뢰를 하지 않는다.

### Issuer/ClusterIssuer 

certificate signing request(csr)을 준수하여 서명된 인증서를 발급해주는 CA이고, k8s리소스 이다.

 

- selfsigned-issuer 생성 (실제 초기 CA 인증서를 부트스트랩해줄 issuer) 

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
	labels:
		app.kubernetes.io/instance: traefik
		app.kubernetes.io/name: cert-manager 
		app.kubernetes.io/managed-by: tmax-cloud
  name: tmaxcloud-selfsigned
spec:
  selfSigned:{}
```

**tmaxcloud-selfsigned** 라는 이름의 selfsigned issuer를 다음과 같이 생성한다.

- CA 인증서 생성

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
	labels:
		app.kubernetes.io/instance: traefik
		app.kubernetes.io/name: cert-manager 
		app.kubernetes.io/managed-by: tmax-cloud
  name: tmaxcloud-ca
spec:
  secretName: **tmaxcloud-ca**
	duration: 8760h0m0s #360d
  renewBefore: 720h0m0s #30d
  commonName: tmaxcloud
  isCA: true
  usages:
		- digital signature
		- key encipherment
		- server auth
		- client auth
	issuerRef:
		kind: ClusterIssuer
		group: cert-manager.io
		name: **tmaxcloud-selfsigned**
```

tmaxcloud-selfsigned가 발급한 CA 인증서가 생성되며, 해당 CA의 key,crt는 tmaxcloud-ca라는 secret 에 저장이 된다.

- CA Issuer 생성

  - 각 모듈에 selfsigned 인증서를 실제로 발급해줄 clusterIssuer를 생성한다.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
	labels:
		app.kubernetes.io/instance: traefik
		app.kubernetes.io/name: cert-manager 
		app.kubernetes.io/managed-by: tmax-cloud
  name: tmaxcloud-issuer
spec:
	  ca: 
			secretName: **tmaxcloud-ca**
```

- 인증서 생성
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
spec:
  dnsNames:
  - {도메인 명}
  isCA: false
  issuerRef:
    name: **tmaxcloud-ca**
  secretName: {인증서를 생성할 시크릿명}
```
