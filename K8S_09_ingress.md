# 9. Ingress 리소스

## 9.1 🔀 Ingress란?
>* Kubernetes에서 Ingress는 클러스터 외부에서 내부 서비스로의 HTTP 및 HTTPS 요청을 라우팅하기 위한 API 객체입니다.
>* 즉, 사용자가 웹 브라우저로 접속할 때, 요청을 적절한 서비스로 전달해주는 진입 지점(입구) 역할을 합니다.
>* http, https등 네트워크 Layer7에 대한 설정을 담담하는 서비스

### 📌 핵심 개념
| 개념                     | 설명                                                       |
| ---------------------- | -------------------------------------------------------- |
| **Ingress**            | 외부 트래픽을 내부 서비스로 라우팅하기 위한 Kubernetes 리소스                  |
| **Ingress Controller** | Ingress 리소스를 해석하고 실제로 동작하게 하는 컨트롤러 (예: NGINX, Traefik 등) |
| **Ingress Rule**       | 도메인/경로에 따라 트래픽을 어느 서비스로 보낼지 정의                           |

### 🧭 동작 흐름
```text
외부 사용자
   │
   ▼
Ingress Controller  ⇐⇐⇐ Ingress Object
   │
   ▼
Service → Pod
```

### 📄 Ingress 예제
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

### 🔧 Ingress Controller 설치
* Ingress 리소스는 단독으로 작동하지 않습니다. 이를 실제로 처리할 Ingress Controller가 있어야 합니다. 예:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-ingress ingress-nginx/ingress-nginx
```
### 🌐 주요 Ingress Controller
| Controller    | 설명                            |
| ------------- | ----------------------------- |
| NGINX Ingress | 가장 널리 사용되는 Ingress Controller |
| Traefik       | 경량, 빠른 설정, 대시보드 제공            |
| HAProxy       | 고성능 프록시                       |
| Istio Gateway | 서비스 메쉬 통합용                    |

### ✅ 요약
* Ingress는 외부 요청 → 내부 서비스 연결을 담당
* 도메인 기반 또는 경로 기반으로 라우팅
* 작동하려면 반드시 Ingress Controller가 필요
* NGINX가 가장 많이 쓰임

### 9.1.2 NGINX Ingress Controller 설치
>* NGINX Ingress Controller는 Kubernetes에서 생성한 Ingress 리소스를 해석하고, 실제로 HTTP(S) 요청을 해당 서비스로 라우팅해주는 컨트롤러입니다.
>* Ingress만으로는 아무 일도 하지 않고, Ingress Controller가 있어야만 외부 트래픽이 내부로 전달됩니다.

#### 🚀 왜 필요할까?
* Kubernetes 서비스는 내부 IP로만 접근 가능 → 외부에서 접근할 방법이 필요
* LoadBalancer는 비용이 크고 경직됨
* NGINX Ingress Controller는 하나의 엔드포인트로 여러 서비스 라우팅 가능

#### 🧱 구성 요소
| 구성 요소              | 설명                                  |
| ------------------ | ----------------------------------- |
| Ingress Controller | Ingress 리소스를 감시하고 NGINX 설정을 동적으로 적용 |
| Ingress 리소스        | 도메인/경로 → 서비스로의 규칙 정의                |
| Service & Pod      | 실제 요청이 전달될 백엔드 앱                    |

#### 📦 설치 (Helm 사용)
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx

# 설치 후 확인:
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

#### 🧾 기본 Ingress 리소스 예시
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
# 이 설정은 myapp.local 도메인으로 들어오는 모든 요청을 myapp-service로 전달합니다.
```

#### ⚙️ 주요 Annotation 예시
| Annotation               | 설명                    |
| ------------------------ | --------------------- |
| `rewrite-target`         | 요청 경로를 백엔드에 전달할 때 재작성 |
| `ssl-redirect`           | HTTP → HTTPS 강제 리다이렉트 |
| `whitelist-source-range` | 특정 IP만 접근 허용          |

#### 🔍 Ingress Controller 동작 확인
```bash
kubectl describe ingress demo-ingress
kubectl logs <nginx-ingress-controller-pod> -n ingress-nginx
```

#### 📌 요약
* NGINX Ingress Controller는 Ingress를 실제로 동작시키는 핵심 구성요소
* Helm으로 쉽게 설치 가능
* 다양한 도메인/경로 기반 라우팅 설정 지원
* Annotation을 활용해 세부 설정 가능

#### ✅ 실습

```bash
# NGINX Ingress Controller를 위한 네임스페이스를 생성합니다.
kubectl create ns ctrl
# namespace/ctrl created
kubectl get ns

# nginx-ingress 설치
#   -n ctrl --create-namespace : ns ctrl 을 직접 생성할 경우 
helm install ingress bitnami/nginx-ingress-controller \
  --version 9.3.12 -n ctrl 
  # -n ctrl --create-namespace

# NAME: ingress
# LAST DEPLOYED: Wed Mar 11 13:31:14 2020
# NAMESPACE: ctrl
# STATUS: deployed
# REVISION: 1
# TEST SUITE: None
# NOTES:
#     ...
```
##### ▶️ 생성되는 Pod: 2개
| Pod 이름 예시                                | 역할                                     |
| ---------------------------------------- | -------------------------------------- |
| `ingress-nginx-ingress-controller-xxxxx` | **Ingress Controller** Pod – 요청 라우팅 처리 |
| `ingress-nginx-ingress-default-backend`  | **기본 백엔드** – 정의되지 않은 경로 처리 (404 응답)    |

```bash
kubectl get pod -n ctrl
# NAME                            READY   STATUS      RESTARTS  AGE
# ingress-controller-7444984      1/1     Running     0         6s
# ingress-default-backend-659bd6  1/1     Running     0         6s
 
kubectl get svc -n ctrl
# NAME                     TYPE          ...  EXTERNAL-IP    PORT(S)  
# ingress-default-backend  ClusterIP     ...  <none>         80/TCP
# ingress-controller       LoadBalancer  ...  10.0.1.1       80:..,443:..
```


## 9.2 `Ingress` 기본 사용법

### 9.2.1 도메인 주소 테스트
>Ingress는 Layer 7 통신이기 때문에 도메인 주소가 있어야 테스트를 할 수 있다.

#### 🔍 sslip.io란?
* sslip.io는 다음과 같은 형식의 도메인을 자동으로 IP로 매핑해줍니다:
* https://sslip.io의 서브 도메인에 IP를 입력하면 해당하는 IP를 DNS lookup 결과로 반환해 줍니다.
* 🧪 장점
  1. 로컬 테스트에 매우 유용
  1. 별도의 DNS 설정 없이 즉시 도메인 사용 가능
  1. 인증서 테스트에도 활용 가능 (예: Let's Encrypt)

```
IP == IP.sslip.io
```

```bash
nslookup 192.168.164.130.sslip.io
# Server:         127.0.0.53
# Address:        127.0.0.53#53

# Non-authoritative answer:
# Name:   192.168.164.130.sslip.io
# Address: 192.168.164.130
```

```bash
# Ingress기능중 하나인 Domain-based routing을 위해 2차 도메인주소 테스트
nslookup subdomain.192.168.164.130.sslip.io
# Server:         127.0.0.53
# Address:        127.0.0.53#53

# Non-authoritative answer:
# Name:   subdomain.192.168.164.130.sslip.io
# Address: 192.168.164.130
```

#### Ingress Controller IP 확인 방법 
* 다음 명령을 통해 Ingress Controller IP를 확인할 수 있습니다. 호스트 서버(마스터, 워커) 중 하나의 내부IP가 반환될 것입니다.

```bash
kubectl get svc -nctrl ingress-nginx-ingress-controller
kubectl get svc -nctrl ingress-nginx-ingress-controller -o json

# Ingress Controller의 외부 IP 주소를 추출하여 변수에 저장
INGRESS_IP=$(kubectl get svc -nctrl ingress-nginx-ingress-controller -ojsonpath="{.status.loadBalancer.ingress[0].ip}")
echo $INGRESS_IP
# 10.0.1.1
```

### 9.2.2 첫 `Ingress` 생성

```bash
kubectl run mynginx --image nginx --expose --port 80
# pod/mynginx created
# pod/service created
```
##### 🔧 명령어 설명
| 항목              | 설명                                         |
| --------------- | ------------------------------------------ |
| `kubectl run`   | Pod을 생성하는 명령                               |
| `mynginx`       | 생성할 Pod의 이름                                |
| `--image nginx` | 사용할 컨테이너 이미지 (여기선 nginx 공식 이미지)            |
| `--expose`      | 이 옵션을 지정하면, 생성된 Pod에 대해 자동으로 Service도 생성   |
| `--port 80`     | 컨테이너가 사용하는 포트 (Service의 `targetPort`로 설정됨) |

```bash
# comma로 여러 리소스를 한번에 조회할 수 있습니다.
# comma사이에 공란있으면 에러 반드시 붙여서 실행할 것
kubectl get pod,svc mynginx
# NAME          READY   STATUS    RESTARTS   AGE
# pod/mynginx   1/1     Running   0          8m38s

# NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# service/mynginx   ClusterIP   10.43.133.146   <none>        80/TCP    14h

vi K8S/CH05/mynginx-ingress.yaml
```
```yaml
# mynginx-ingress.yaml
apiVersion: networking.k8s.io/v1       # Ingress 리소스의 최신 API 버전
kind: Ingress                          # 리소스 종류: Ingress
metadata:
  name: mynginx                        # Ingress 리소스의 이름
  annotations:
    kubernetes.io/ingress.class: nginx # 사용하려는 Ingress Controller 명시 (여기서는 nginx)
spec:
  rules:
  - host: 192.168.164.130.sslip.io            # 요청을 처리할 도메인 (sslip.io는 IP 기반 도메인 생성 서비스)
    http:
      paths:
      - path: /                        # URL 경로
        pathType: Prefix               # 경로 매칭 방식: 접두사(prefix)로 매칭
        backend:
          service:
            name: mynginx             # 연결할 서비스 이름 (kubectl get svc mynginx로 확인 가능)
            port:
              number: 80              # 서비스의 포트 번호
```

```bash
kubectl apply -f K8S/CH05/mynginx-ingress.yaml
# ingress.extensions/mynginx created

kubectl get ingress
# NAME      CLASS    HOSTS                      ADDRESS           PORTS   AGE
# mynginx   <none>   192.168.164.130.sslip.io   192.168.164.130   80      14h

kubectl get svc -n ctrl ingress-nginx-ingress-controller
# NAME                               TYPE           CLUSTER-IP      EXTERNAL-IP                       PORT(S)                      AGE
# ingress-nginx-ingress-controller   LoadBalancer   10.43.112.135   192.168.164.130,192.168.164.131   80:31842/TCP,443:31061/TCP   15h

# mynginx 서비스로 연결
# curl <EXTERNAL-IP>.sslip.io
curl 192.168.164.130.sslip.io
curl http://192.168.164.130.sslip.io
curl http://192.168.164.131.sslip.io
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

### 9.2.3 도메인 기반 라우팅
>* 도메인 기반 라우팅은 접속한 도메인 이름에 따라 다른 서비스로 요청을 전달하는 방식입니다. 
>* 즉, 하나의 Ingress Controller가 여러 도메인 이름을 감지해서 요청을 해당 서비스로 연결해주는 기능
>* 쿠버네티스에서는 주로 Ingress를 통해 구현하며, 예를 들어 아래와 같이 동작합니다:
>* 예: http://app1.example.com  → 서비스 A, http://app2.example.com  → 서비스 B

```bash
# apache web server
kubectl run apache --image httpd --expose --port 80
# pod/apache created
# service/apache created

# nginx web server
kubectl run nginx --image nginx --expose --port 80
# pod/nginx created
# service/nginx created

vi  K8S/CH05/domain-based-ingress.yaml
```

```yaml
# domain-based-ingress.yaml
# Apache 도메인을 위한 Ingress 정의
apiVersion: networking.k8s.io/v1    # Ingress 리소스에 사용하는 최신 API 버전
kind: Ingress                       # 리소스 종류: Ingress
metadata:
  name: apache-domain               # Ingress 리소스의 이름
  annotations:
    kubernetes.io/ingress.class: nginx  # 사용하려는 Ingress Controller를 지정 (nginx ingress controller)

spec:
  rules:                            # 라우팅 규칙 정의
  - host: apache.192.168.164.130.sslip.io  # 요청을 받을 도메인 이름. sslip.io는 IP 기반 도메인 매핑 서비스
    http:
      paths:
      - path: /                     # URL 경로. 여기서는 루트(/) 경로로 접근 시
        pathType: Prefix            # 경로 매칭 방식: 접두사 매칭 ("/"로 시작하면 모두 해당됨)
        backend:
          service:                  # 요청을 전달할 대상 서비스 정의
            name: apache            # 연결할 Kubernetes 서비스 이름 (kubectl get svc로 확인 가능)
            port:
              number: 80            # 해당 서비스가 노출하고 있는 포트 번호

---

# Nginx 도메인을 위한 Ingress 정의
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-domain
  annotations:
    kubernetes.io/ingress.class: nginx

spec:
  rules:
  - host: nginx.192.168.164.130.sslip.io   # nginx 서비스용 도메인 이름
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx             # 연결할 nginx 서비스 이름
            port:
              number: 80            # nginx 서비스가 노출 중인 포트

# 🔍 참고 설명
# sslip.io는 특정 IP를 포함한 도메인명을 자동으로 그 IP로 매핑해주는 서비스입니다.
# 예: apache.10.0.1.1.sslip.io → 10.0.1.1
# pathType: Prefix는 지정한 경로로 시작하는 모든 요청을 의미합니다.
# 예: /hello, /hello/world 모두 /에 매칭됨.
# Ingress Controller는 반드시 설치되어 있어야 하며, 일반적으로 nginx나 traefik이 사용됩니다.
# 위 설정은 Ingress가 apache와 nginx라는 두 서비스로 요청을 각각 라우팅해주는 역할을 합니다.            
```

```bash
kubectl apply -f K8S/CH05/domain-based-ingress.yaml
# ingress.extensions/apache-domain created
# ingress.extensions/nginx-domain created

curl apache.192.168.164.130.sslip.io
# <html><body><h1>It works!</h1></body></html>

curl nginx.192.168.164.130.sslip.io
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

### 9.2.4 Path 기반 라우팅
>* Path 기반 라우팅(Path-based Routing)은 하나의 도메인 이름 아래에서 **요청 경로(path)**에 따라 서로 다른 서비스로 트래픽을 분기하는 방법입니다. 
>* 예를 들어 다음과 같은 요청들을 각각 다른 백엔드로 라우팅할 수 있습니다:
>* http://example.com/app1 → app1 서비스
>* http://example.com/app2 → app2 서비스
#### 📌 주요 개념 설명
| 필드                            | 설명                                     |
| ----------------------------- | -------------------------------------- |
| `host`                        | 공통 도메인 이름 (sslip.io 사용 시 IP 기반 도메인 가능) |
| `path`                        | 요청 경로 조건 (`/app1`, `/app2`)            |
| `pathType: Prefix`            | 해당 경로로 시작하는 모든 요청을 처리                  |
| `backend.service.name`        | 요청을 전달할 Kubernetes 서비스 이름              |
| `backend.service.port.number` | 서비스가 열고 있는 포트                          |

#### 🌐 동작 예시
* 요청: http://example.192.168.1.100.sslip.io/app1 → app1 서비스로 전달
* 요청: http://example.192.168.1.100.sslip.io/app2/info → app2 서비스로 전달

#### ⚠️ 주의사항
* 반드시 nginx Ingress Controller가 클러스터에 설치되어 있어야 합니다.
* sslip.io는 테스트용 IP 기반 도메인 매핑 도구입니다. 실제 운영 환경에서는 실 도메인을 사용해야 합니다.
* 모든 대상 서비스 (app1, app2)는 이미 클러스터 내에 ClusterIP 또는 NodePort 형태로 배포되어 있어야 합니다.

```bash
vi  K8S/CH05/path-based-ingress.yaml
```
```yaml
# path-based-ingress.yaml
# apache-path Ingress 설정
apiVersion: networking.k8s.io/v1               # 최신 Ingress API 버전
kind: Ingress                                  # 리소스 종류: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx         # 사용할 Ingress Controller (nginx)
    nginx.ingress.kubernetes.io/rewrite-target: /  # /apache → / 로 rewrite 처리
  name: apache-path                            # Ingress 리소스 이름
spec:
  rules:
  - host: 192.168.164.130.sslip.io             # 요청을 받을 도메인 (IP 기반 도메인)
    http:
      paths:
      - path: /apache                          # 요청 경로가 /apache 일 때
        pathType: Prefix                       # /apache로 시작하는 모든 요청 처리
        backend:
          service:
            name: apache                       # apache 서비스로 라우팅
            port:
              number: 80                       # apache 서비스의 포트
---
# nginx-path Ingress 설정
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /  # /nginx → / 로 rewrite 처리
  name: nginx-path
spec:
  rules:
  - host: 192.168.164.130.sslip.io
    http:
      paths:
      - path: /nginx                          # 요청 경로가 /nginx 일 때
        pathType: Prefix
        backend:
          service:
            name: nginx                       # nginx 서비스로 라우팅
            port:
              number: 80
```
##### 🔍 핵심 개념 요약
| 항목                        | 설명                                                        |
| ------------------------- | --------------------------------------------------------- |
| `rewrite-target: /`       | `/apache`, `/nginx` 경로를 내부에서는 `/`로 변환하여 서비스가 루트에서 동작하도록 함 |
| `host: 10.0.1.1.sslip.io` | IP 기반 도메인 (sslip.io는 `10.0.1.1` IP에 대해 도메인처럼 작동)          |
| `path: /apache`, `/nginx` | 각각의 경로로 들어오는 요청을 다른 서비스로 분기                               |
| `service.name`            | 실제 트래픽이 전달될 Kubernetes 서비스 이름                             |
| `service.port.number`     | 해당 서비스가 노출한 포트 번호                                         |

```bash
kubectl apply -f K8S/CH05/path-based-ingress.yaml
# ingress.extensions/apache-path created
# ingress.extensions/nginx-path created

curl 192.168.164.130.sslip.io/apache
# <html><body><h1>It works!</h1></body></html>

curl 192.168.164.130.sslip.io/nginx
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

## 9.3 Basic Auth 설정

### 9.3.1 Basic Authentication
* HTTP 프로토콜에는 자체적으로 인증을 위한 메커니즘이 설계되어 있다.
* 그 중에 Basic Authentication은 간단한 user id, password를 base64로 인코딩하여 HTTP heade로 전달하여 인증처리
* user=foo, pw=bar로 basic auth처리하는 페이지 : https://httpbin.org/basic-auth/foo/bar

```bash
Authorization: Basic $BASE64(user:password)
```

```bash
# 헤더 없이 접속 시도
# -v : verbose 모드 (요청과 응답의 헤더 등 자세한 정보 출력)
curl -v https://httpbin.org/basic-auth/foo/bar
# HTTP/2 401 
# ...
# www-authenticate: Basic realm="Fake Realm"

echo -n foo:bar | base64

# -H : 수동 헤더 방식, Base64 인코딩값 직접 지정
# basic auth 헤더 전송
curl -v -H "Authorization: Basic $(echo -n foo:bar | base64)" https://httpbin.org/basic-auth/foo/bar
# HTTP/2 200 
# ..
# {
#   "authenticated": true, 
#   "user": "foo"
# }
```

### 9.3.2 Basic Auth 설정

```bash
# apache2-utils 설치 시 포함되는 유틸리티: htpasswd – 기본 인증용 사용자명/비밀번호 파일 생성 및 관리 툴
sudo apt install -y apache2-utils

# 아이디는 foo, 비밀번호는 bar인 auth 파일 생성
# htpasswd HTTP Basic 인증에서 사용할 사용자명/비밀번호 파일 (.htpasswd)을 생성하거나 수정하는 도구입니다 
# -c (create)
#  ... 지정한 파일이 없으면 새로 생성하고, 이미 존재하는 파일이 있으면 덮어쓰기 합니다.
#  ... 예: htpasswd -c auth foo: 파일 auth를 만들고 user foo를 등록합니다 
# -b (batch mode)
#  ... 비밀번호를 명령행 인자로 직접 전달합니다.
#  ... Without -b, htpasswd 실행 시 **비밀번호를 직접 입력(prompt)**해야 합니다.
#  ... -b를 사용하면 htpasswd -b auth foo bar는 bar를 직접 읽어 처리합니다 .
# auth : 생성될 파일 이름입니다. 이후에 이 파일을 Kubernetes Secret으로 만들어 인증에 사용합니다.
# foo : 생성할 사용자 이름(username)입니다.
# bar : 해당 사용자의 비밀번호(password)입니다.
htpasswd -cb auth foo bar
cat auth

# 생성한 auth 파일을 Secret으로 생성합니다.
# kubectl create secret generic : Opaque 타입 Secret(임의 데이터 저장 형태)을 생성
# basic-auth : 생성될 Secret의 이름
# --from-file=auth : 현재 디렉토리의 auth 파일 내용을 auth라는 key로 Secret에 저장
kubectl create secret generic basic-auth --from-file=auth

# Secret 리소스 생성 확인
kubectl get secret basic-auth -oyaml
# apiVersion: v1
# data:
#   auth: Zm9vOiRhcHIxJE1UTy9MMUN0JEdNek8xOVZtMXdKYWt6R0tjLjhQTS8K
# kind: Secret
# metadata:
#   name: basic-auth
#   namespace: default
#   resourceVersion: "3648288"
#   selfLink: /api/v1/namespaces/default/secrets/basic-auth
#   uid: b9966176-5259-4e3f-8476-c7e308ae21a1
# type: Opaque

vi K8S/CH05/apache-auth.yaml
```
```yaml
# apache-auth.yaml
# apache-auth Ingress 정의: Basic Auth 적용
apiVersion: networking.k8s.io/v1                # ✅ 최신 Ingress API 버전 사용
kind: Ingress                                   # 리소스 타입: Ingress
metadata:
  name: apache-auth                             # Ingress 리소스 이름
  annotations:
    kubernetes.io/ingress.class: nginx          # 이 Ingress를 NGINX Controller가 처리하도록 지정
    nginx.ingress.kubernetes.io/auth-type: basic # 인증 방식: Basic Authentication 사용
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # 인증에 사용할 Secret 이름 (이름: basic-auth, key: auth)
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
    # 인증 창에 표시될 문구
spec:
  rules:
  - host: apache-auth.192.168.164.130.sslip.io
    # 요청할 도메인 (sslip.io 기반 IP->도메인 자동 매핑)
    http:
      paths:
      - path: /                                  # 경로 지정 (루트)
        pathType: Prefix                         # prefix 매칭 방식 사용 (이 경로로 시작하는 모든 요청)
        backend:
          service:                               # 최신 형식으로 backend 서비스 지정 (networking.k8s.io/v1)
            name: apache                         # 이 서비스 이름으로 라우팅
            port:
              number: 80                         # 이 포트로 요청 전달
```

```bash
kubectl apply -f K8S/CH05/apache-auth.yaml
# ingress.extensions/apache-auth created

curl -I apache-auth.192.168.164.130.sslip.io
# HTTP/1.1 401 Unauthorized
# Server: nginx/1.17.10
# Date: Tue, 07 Jul 2020 12:30:43 GMT
# Content-Type: text/html
# Content-Length: 180
# Connection: keep-alive
# WWW-Authenticate: Basic realm="Authentication Required - foo"

# 🔍 -I 또는 --head   : 서버로 HEAD 요청을 보내고, 응답 헤더만 가져옵니다.
# 🔧 -H 또는 --header : 사용자 정의 HTTP 헤더를 요청에 포함시킬 때 사용
curl -I -H "Authorization: Basic $(echo -n foo:bar | base64)" apache-auth.192.168.164.130.sslip.io
# HTTP/1.1 200 OK
# Server: nginx/1.17.10
# Date: Tue, 07 Jul 2020 12:31:14 GMT
# Content-Type: text/html
# Content-Length: 45
# Connection: keep-alive
# Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
# ETag: "2d-432a5e4a73a80"
# Accept-Ranges: bytes
```
##### ✅ 비교 요약
| 옵션   | 설명                 | 출력         | 사용 예                        |
| ---- | ------------------ | ---------- | --------------------------- |
| `-I` | HEAD 요청, 응답 헤더만 출력 | 응답 헤더만 보여줌 | `curl -I http://site`       |
| `-H` | 요청에 헤더 추가          | 응답 본문도 포함됨 | `curl -H "Origin: ..." URL` |


## 9.4 TLS 설정
>* TLS(Transport Layer Security)는 인터넷 통신을 암호화하고 데이터 보안을 보장하기 위한 프로토콜
>* 웹사이트 주소가 **https://로 시작할 때 사용되는 기술**이며, 이전 버전인 SSL(Secure Sockets Layer)의 후속 버전

### 🔐 TLS의 주요 기능   
1. 암호화(Encryption) : 전송 중인 데이터를 암호화해서 중간에 누군가가 가로채더라도 내용을 알 수 없도록 보호합니다.
2. 무결성(Integrity) : 데이터가 전송 중에 변경되거나 위조되지 않았음을 검증합니다.
3. 인증(Authentication) 
   - 통신하는 두 당사자(예: 웹 브라우저와 서버)가 서로를 신뢰할 수 있는지 확인합니다.
   - 일반적으로 **서버 인증서(SSL 인증서)**를 사용하여 웹사이트의 신원을 증명합니다.

### 🔧 TLS 동작 방식 (핸드셰이크 과정)
1. 클라이언트 Hello : 클라이언트가 서버에게 "나 TLS 쓸 거야!"라며 초기 정보(지원하는 암호화 방식, TLS 버전 등)를 보냅니다.
2. 서버 Hello : 서버는 클라이언트의 요청을 보고 자신이 선택한 암호화 방식, 인증서(서버의 신원), 공개 키 등을 보냅니다.
3. 키 교환 : 클라이언트는 서버의 공개 키를 사용해 암호화된 세션 키를 보냅니다.(이 세션 키는 이후 대화에 사용됩니다 (속도를 높이기 위해 대칭 키 사용))
4. 세션 시작 : 양쪽 모두 같은 세션 키를 갖고 있으므로, 이후에는 이 키로 데이터를 암호화하여 주고받습니다.

### 9.4.1 Self-signed 인증서 설정
#### 🔐 Self-signed 인증서란?
>* Self-signed 인증서는 이름 그대로 자체적으로 서명한 SSL/TLS 인증서입니다. 
>* 일반적으로 인증서는 **신뢰된 인증 기관(CA; Certificate Authority)**가 서명해야 웹 브라우저나 운영 체제가 "신뢰할 수 있는 사이트"라고 인정합니다.
>* 하지만 Self-signed 인증서는 CA를 거치지 않고, 사용자가 직접 생성하고 서명한 인증서

#### 🧪 Self-signed 인증서가 사용되는 경우
* 개발 또는 테스트 환경
* 사내 서비스에서 외부 접속이 없는 경우
* VPN이나 사설 네트워크 안에서 TLS 통신을 할 때
* 비용을 아끼고 싶을 때 (공인 인증서 발급은 유료일 수 있음)

#### ⚠️ Self-signed 인증서의 단점
1. 신뢰성 부족
  - 브라우저는 "이 인증서는 신뢰되지 않음"이라고 경고를 표시합니다.
  - 예: Chrome에서 "이 연결은 안전하지 않음" 같은 경고 화면
2. MITM 공격에 취약
   - 중간자 공격(Man-In-The-Middle)을 방지할 수 없음. 인증서를 위조해도 브라우저가 막지 않기 때문
3. 자동 검증 실패
   - 사용자나 시스템이 별도로 인증서를 수동으로 신뢰 목록에 등록해야 함

#### Self-signed 인증서 생성하기

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=apache-tls.192.168.164.130.sslip.io"
# Generating a RSA private key
# ...................................................+++++
# .....+++++
# writing new private key to 'tls.key'
# -----
cat tls.crt  # 인증서
cat tls.key  # 개인키
```
#### 🧩 옵션별 상세 설명
| 옵션                                                | 의미                                                                                                                |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `openssl req`                                     | 인증서 서명 요청(Certificate Signing Request, CSR)을 생성하는 명령입니다.                                                          |
| `-x509`                                           | CSR 대신 직접 \*\*self-signed 인증서(X.509)\*\*를 생성합니다. 즉, `req` 명령이지만 CSR 파일을 만들지 않고 곧바로 인증서를 만듭니다.                     |
| `-nodes`                                          | "No DES"의 줄임말. 개인 키를 암호화하지 않고 생성합니다. <br>즉, 인증서 사용할 때 **비밀번호 입력 없이 자동 사용**할 수 있도록 만듭니다.                           |
| `-days 365`                                       | 인증서 유효 기간을 365일(1년)로 설정합니다.                                                                                       |
| `-newkey rsa:2048`                                | 새로운 RSA 키를 생성하며, **2048비트 길이**의 키를 사용합니다.                                                                         |
| `-keyout tls.key`                                 | 생성한 **개인 키를 저장할 파일 이름**입니다.                                                                                       |
| `-out tls.crt`                                    | 생성한 **인증서(.crt)를 저장할 파일 이름**입니다.                                                                                  |
| `-subj "/CN=apache-tls.192.168.164.130.sslip.io"` | 인증서의 "주체(subject)" 정보를 명시합니다.<br>여기서 `CN`은 **Common Name**, 즉 도메인 이름입니다.<br>브라우저는 인증서의 CN이 접속하려는 주소와 일치하는지 검사합니다. |

#### 🔎 -subj에 들어간 내용 해석
* CN은 Common Name의 약자로, 보통은 도메인 이름이 들어갑니다.
* apache-tls.192.168.164.130.sslip.io는 SSL 인증서가 유효하다고 주장하는 도메인입니다.
  - sslip.io는 IP 주소를 도메인처럼 사용할 수 있게 해주는 무료 DNS 서비스입니다.
  - 예: 192.168.164.130.sslip.io는 자동으로 192.168.164.130에 매핑됩니다.

#### 📁 결과 파일들
| 파일 이름     | 설명                              |
| --------- | ------------------------------- |
| `tls.key` | 개인 키 (서버가 소중히 보관해야 함)           |
| `tls.crt` | self-signed 공개 인증서 (클라이언트가 참조함) |

#### ✅ 요약
1. 이 명령은 localhost 또는 사설망에서 TLS 테스트를 위해 다음을 수행합니다:
2. 2048비트 RSA 개인 키 생성 (tls.key)
3. 1년짜리 self-signed 인증서 생성 (tls.crt)
4. CN이 apache-tls.192.168.164.130.sslip.io인 인증서 생성
5. 인증서에 비밀번호 없이 자동 사용할 수 있도록 구성

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: my-tls-certs
  namespace: default
data:
  tls.crt: $(cat tls.crt | base64 | tr -d '\n')
  tls.key: $(cat tls.key | base64 | tr -d '\n')
type: kubernetes.io/tls
EOF
```

#### Ingress TLS 설정하기
```bash
vi K8S/CH05/apache-tls.yaml
```
```yaml
# apache-tls.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-tls
spec:
  tls:
    - hosts:
        - apache-tls.192.168.164.130.sslip.io
      secretName: my-tls-certs
  rules:
    - host: apache-tls.192.168.164.130.sslip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: apache
                port:
                  number: 80
```
##### 🔍 주요 필드 설명
| 필드                                 | 설명                                        |
| ---------------------------------- | ----------------------------------------- |
| `apiVersion: networking.k8s.io/v1` | Kubernetes 1.19+에서 사용하는 Ingress 최신 API 버전 |
| `tls.hosts`                        | HTTPS 인증이 적용될 도메인 이름                      |
| `tls.secretName`                   | 위 도메인의 TLS 인증서가 저장된 Secret 이름             |
| `rules.host`                       | 실제 Ingress가 라우팅할 도메인                      |
| `http.paths`                       | 경로별 서비스 라우팅 정의                            |
| `pathType: Prefix`                 | `/`로 시작하는 모든 경로를 포함                       |
| `backend.service.name`             | 연결할 Service 이름 (`apache`)                 |
| `backend.service.port.number`      | 연결할 Service 포트 (예: 80)                    |


```bash
kubectl apply -f K8S/CH05/apache-tls.yaml
# ingress.networking.k8s.io/apache-tls created

kubectl get ingress apache-tls
# NAME            CLASS    HOSTS                                  ADDRESS           PORTS     AGE
# apache-tls      <none>   apache-tls.192.168.164.130.sslip.io                      80, 443   25s

```

### 9.4.2 cert-manager를 이용한 인증서 발급 자동화
>* Kubernetes 클러스터에서 TLS 인증서를 자동으로 발급하고 갱신할 수 있다. 
>* 주로 Let's Encrypt 등 무료 공개 CA와 연동하거나 사설 CA와 연동해 자동화할 수 있다.

#### cert-manager 설치

```bash
# cert-manager라는 네임스페이스 생성
kubectl create namespace cert-manager

# cert-manager 관련 사용자 정의 리소스 생성
kubectl apply --validate=false -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

# customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
# customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manage...
# ...

# jetstack 레포지토리 추가
helm repo add jetstack https://charts.jetstack.io
#"jetstack" has been added to your repositories

# 레포지토리 index 업데이트
helm repo update
# Hang tight while we grab the latest from your chart repositories...
# ...Successfully got an update from the "jetstack" chart repository

# cert-manager 설치
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true
```
##### ✅ 명령어 설명
| 옵션                         | 설명                                                          |
| -------------------------- | ----------------------------------------------------------- |
| `cert-manager`             | Helm 릴리스 이름입니다. 이후 Helm upgrade/remove 등에 사용됩니다.            |
| `jetstack/cert-manager`    | 설치할 Helm 차트 (Jetstack 리포에서 가져옴)                             |
| `--namespace cert-manager` | cert-manager 리소스를 설치할 Kubernetes 네임스페이스                     |
| `--create-namespace`       | 지정한 네임스페이스가 없으면 자동 생성합니다.                                   |
| `--set installCRDs=true`   | CRD(Custom Resource Definitions)도 자동 설치되도록 설정합니다. 매우 중요합니다! |


#### Issuer 생성
* cert-manager를 설치하셨다면 이제 인증서를 자동으로 발급해주는 Issuer 또는 ClusterIssuer를 생성할 수 있다.
* 보통은 ClusterIssuer를 추천합니다. 하나만 설정하면 어떤 네임스페이스에서도 TLS 자동 발급이 가능하기 때문이다.
* ingress의 설정값을 참조하여 Let's Encrypt(https://letsencrypt.org/)사이트에 정식 인증서를 요청

##### 🌐 Let's Encrypt란?
| 항목     | 설명                                                                          |
| ------ | ---------------------------------------------------------------------------     |
| 이름     | [Let's Encrypt](https://letsencrypt.org)                                      |
| 운영 주체  | [ISRG (Internet Security Research Group)](https://www.abetterinternet.org/) |
| 인증서 타입 | X.509 TLS 인증서 (도메인 인증형 - DV)                                      |
| 비용     | ✅ **완전 무료**                                                             |
| 유효 기간  | 기본 90일 (자동 갱신 권장)                                                  |
| 자동화 도구 | **certbot**, **cert-manager**, **acme.sh** 등                              |

##### 🧾 Issuer vs ClusterIssuer
| 항목              | 설명                           |
| --------------- | ---------------------------- |
| `Issuer`        | **네임스페이스 단위**로 유효한 인증서 발급자   |
| `ClusterIssuer` | **클러스터 전체에서 사용 가능**한 인증서 발급자 |

```bash
vi K8S/CH05/http-issuer.yaml
```
```yaml
# http-issuer.yaml
apiVersion: cert-manager.io/v1              # 최신 API 버전 (v1alpha2는 deprecated)
kind: ClusterIssuer                         # 클러스터 전체에서 사용할 수 있는 인증서 발급자
metadata:
  name: http-issuer                         # ClusterIssuer의 이름 (Ingress에서 참조함)
spec:
  acme:                                     # ACME(Automatic Certificate Management Environment) 방식 사용
    email: iwbaek@gmail.com                 # 인증서 갱신 실패 등 알림을 받을 이메일 주소(반드시 실제주소)
    server: https://acme-v02.api.letsencrypt.org/directory   # Let's Encrypt 프로덕션 서버 주소
    privateKeySecretRef:
      name: issuer-key                      # cert-manager가 생성하는 ACME 계정 비밀키를 저장할 Secret 이름
    solvers:                                # 도메인 소유권을 인증하기 위한 방식 정의
    - http01:                               # HTTP-01 챌린지 방식 사용 (Ingress 기반)
        ingress:
          class: nginx                      # 사용할 Ingress Controller 클래스 (ex: nginx, traefik 등)
```
##### ⚠️ 필수 조건
| 조건                               | 설명                                                                               |
| -------------------------------- | -------------------------------------------------------------------------------- |
| 도메인 이름                           | `apache-tls.example.com` 등 공인 도메인 필요                                             |
| 외부에서 HTTP 접근 가능                  | Let's Encrypt가 `http://<your-domain>/.well-known/acme-challenge/...`로 접근해야 인증 성공 |
| Ingress Controller (예: NGINX) 설치 | cert-manager는 Ingress를 통해 ACME 챌린지를 처리함                                          |
| `nginx` Ingress 클래스 명시           | 위의 `ingress.class: nginx`가 실제 IngressController의 클래스명과 일치해야 함                    |

```bash
kubectl apply -f K8S/CH05/http-issuer.yaml
# clusterissuer.cert-manager.io/http-issuer created

kubectl get clusterissuer
# NAME          READY   AGE
# http-issuer   True    2m
```

#### cert-manager가 관리하는 TLS `Ingress` 생성
##### 🔧 사전 준비 확인 체크리스트
| 항목                                  | 확인 여부                                        |
| ----------------------------------- | -------------------------------------------- |
| ✅ `http-issuer` ClusterIssuer 생성됨    | `kubectl get clusterissuer`                  |
| ✅ Ingress Controller 설치됨 (예: nginx) | `kubectl get pods -n ingress-nginx`          |
| ✅ 도메인 → 외부 IP로 정상 라우팅        | `apache-tls.example.com`이 Ingress IP를 가리켜야 함 |
| ✅ 서비스 이름 `apache` 존재             | `kubectl get svc` 로 확인                       |

```bash
vi K8S/CH05/apache-tls-issuer.yaml
```
```yaml
# apache-tls-issuer.yaml
apiVersion: networking.k8s.io/v1             # 최신 Ingress API 버전
kind: Ingress                                # Ingress 리소스: HTTP(S) 라우팅 설정
metadata:
  name: apache-tls-issuer                    # Ingress 리소스의 이름
  annotations:
    cert-manager.io/cluster-issuer: http-issuer  # cert-manager가 사용할 ClusterIssuer 이름
spec:
  ingressClassName: nginx                    # Ingress Controller 클래스 이름 (예: nginx, traefik 등)
  tls:
    - hosts:
        - apache-issuer.192.168.164.130.sslip.io  # 인증서를 적용할 도메인
      secretName: apache-tls                 # cert-manager가 생성할 TLS Secret 이름
  rules:
    - host: apache-issuer.192.168.164.130.sslip.io  # 수신할 도메인
      http:
        paths:
          - path: /                          # 모든 요청 경로에 대해
            pathType: Prefix                 # 접두사 방식 매칭 (예: `/abc`는 `/abc/def` 포함)
            backend:
              service:
                name: apache                 # 연결할 백엔드 서비스 이름 (웹 서버 등)
                port:
                  number: 80                 # 해당 서비스의 포트 번호
```

```bash
kubectl apply -f K8S/CH05/apache-tls-issuer.yaml
# ingress.networking.k8s.io/apache-tls-issuer created

kubectl get certificate 
# NAME         READY   SECRET       AGE
# apache-tls   False   apache-tls   38s
# True로 될때까지 대기!!

kubectl get certificate 
# NAME         READY   SECRET       AGE
# apache-tls   True    apache-tls   75s
```

### Clean up

```bash
kubectl delete ingress --all
kubectl delete pod apache nginx mynginx
kubectl delete svc apache nginx mynginx

#  cert-manager 전체 삭제
kubectl delete -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

# 🔄 전체 한 번에 정리 스크립트 (선택)
kubectl delete ingress apache-ingress
kubectl delete certificate apache-tls
kubectl delete secret apache-tls my-tls-certs
kubectl delete clusterissuer letsencrypt-http selfsigned-issuer
kubectl delete certificaterequest --all
kubectl delete challenge --all
kubectl delete namespace cert-manager

# 모두 삭제한 후에는 관련된 리소스들이 다음 명령에 나타나지 않아야 합니다:
kubectl get ingress
kubectl get certificate
kubectl get secret
kubectl get clusterissuer
```

<hr>


### ✅ 전제 조건
* kubectl 설치 및 클러스터 연결됨
* apache 서비스가 이미 default 네임스페이스에 존재함
* 외부에서 접근 가능한 IP (예: 192.168.164.131)
* nginx-ingress-controller가 설치되어 있음

### 🧙 전체 설정 스크립트

```bash
#!/bin/bash

# 🧼 이전 리소스 정리
kubectl delete ingress apache-ingress --ignore-not-found
kubectl delete certificate apache-tls --ignore-not-found
kubectl delete secret apache-tls --ignore-not-found
kubectl delete clusterissuer letsencrypt-http --ignore-not-found

echo "✅ 이전 리소스 정리 완료"

# 1️⃣ cert-manager 설치
kubectl apply --validate=false -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
echo "✅ cert-manager 설치 중..."

# cert-manager가 준비될 때까지 대기
echo "⌛ cert-manager Pod 준비 대기 중..."
kubectl wait --for=condition=Available --timeout=120s deployment/cert-manager -n cert-manager
kubectl wait --for=condition=Available --timeout=120s deployment/cert-manager-webhook -n cert-manager

# 2️⃣ Let's Encrypt ClusterIssuer 생성
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-http-private-key
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

echo "✅ ClusterIssuer 생성 완료"

# 3️⃣ TLS Ingress 생성 (apache 서비스용)
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-http
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - apache.192.168.164.131.sslip.io
    secretName: apache-tls
  rules:
  - host: apache.192.168.164.131.sslip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: apache
            port:
              number: 80
EOF

echo "✅ Ingress 생성 완료"

# 4️⃣ 결과 확인 안내
echo
echo "📍 모든 설정이 완료되었습니다."
echo "🔗 https://apache.192.168.164.131.sslip.io 로 접속해 보세요!"
echo "🔎 인증서 상태 확인:"
echo "    kubectl get certificate"
```

### 📌 사용법
* 위 내용을 setup-tls.sh라는 파일로 저장
* 실행 권한 부여: chmod +x setup-tls.sh
* 실행: ./setup-tls.sh

### 💬 커스터마이즈 가능한 부분
| 항목                       | 설명                         |
| ------------------------ | -------------------------- |
| `your-email@example.com` | 실제 본인의 이메일 주소로 변경          |
| `192.168.164.131`        | 실제 외부에서 접근 가능한 IP 주소로 변경   |
| `apache`                 | 서비스 이름이 다른 경우 YAML에서 수정 필요 |

### 🔍 설정 후 확인 명령
```bash
kubectl get certificate
kubectl describe certificate apache-tls
kubectl get ingress
```
### 브라우저 접속:
* https://apache.192.168.164.131.sslip.io