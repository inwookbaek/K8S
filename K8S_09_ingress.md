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

```bash
kubectl get pod -n ctrl
# NAME                            READY   STATUS      RESTARTS  AGE
# ingress-controller-7444984      1/1     Running     0         6s
# svclb-ingress-controller-dcph4  2/2     Running     0         6s
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
nslookup 10.0.1.1.sslip.io
# Address: 10.0.1.1
```

```bash
# Ingress기능중 하나인 Domain-based routing을 위해 2차 도메인주소 테스트
nslookup subdomain.10.0.1.1.sslip.io
# Address: 10.0.1.1
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
>* 쿠버네티스에서는 주로 Ingress를 통해 구현하며, 예를 들어 아래와 같이 동작합니다:

```bash
# apache web server
kubectl run apache --image httpd --expose --port 80
# pod/apache created
# service/apache created

# nginx web server
kubectl run nginx --image nginx --expose --port 80
# pod/nginx created
# service/nginx created
```

```yaml
# domain-based-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: apache-domain
spec:
  rules:
    # apache 서브 도메인
  - host: apache.10.0.1.1.sslip.io
    http:
      paths:
      - backend:
          serviceName: apache
          servicePort: 80
        path: /
---  
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: nginx-domain
spec:
  rules:
    # nginx 서브 도메인
  - host: nginx.10.0.1.1.sslip.io
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /
```

```bash
kubectl apply -f domain-based-ingress.yaml
# ingress.extensions/apache-domain created
# ingress.extensions/nginx-domain created

curl apache.10.0.1.1.sslip.io
# <html><body><h1>It works!</h1></body></html>

curl nginx.10.0.1.1.sslip.io
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

### 9.2.4 Path 기반 라우팅

```yaml
# path-based-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: / 
  name: apache-path
spec:
  rules:
  - host: 10.0.1.1.sslip.io
    http:
      paths:
      - backend:
          serviceName: apache
          servicePort: 80
        path: /apache
---  
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: / 
  name: nginx-path
spec:
  rules:
  - host: 10.0.1.1.sslip.io
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
        path: /nginx
```

```bash
kubectl apply -f path-based-ingress.yaml
# ingress.extensions/apache-path created
# ingress.extensions/nginx-path created

curl 10.0.1.1.sslip.io/apache
# <html><body><h1>It works!</h1></body></html>

curl 10.0.1.1.sslip.io/nginx
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

