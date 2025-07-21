
# ChatGPT - Ingress 구현 절차

## A. Ingress 설정 - http 접속
### 📦 1단계: nginx ingress 설치하기 (문지기 만들기)
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### ➕ 설치 명령:
```bash
helm install ingress bitnami/nginx-ingress-controller \
  --namespace ctrl \
  --create-namespace
💡 Ingress Controller는 "모든 외부 요청을 받아들이는 문지기" 역할이에요.
```

### 🕸️ 2단계: Apache 웹서버 만들기 (서비스 대상 만들기)
```bash
kubectl create deployment apache --image=httpd
kubectl expose deployment apache --port=80 --target-port=80
# 💡 Apache 웹서버는 우리가 접속하고 싶은 "웹사이트" 역할이에요.
```

### 📛 3단계: 내 클러스터의 IP 확인하기
* Ingress Controller가 갖고 있는 외부 IP를 확인합니다:

```bash
kubectl get svc -n ctrl
# 출력 예:
# NAME                       TYPE           EXTERNAL-IP     PORT(S)
# ingress-nginx-controller   LoadBalancer   203.0.113.10    80:... 443:...
# ☑️ 여기서 EXTERNAL-IP가 있다면 복사해 둡니다! → 203.0.113.10 등
```

### 🧾 4단계: Ingress 리소스 만들기 (주소 연결)
* 아래 파일을 apache-ingress.yaml 이름으로 저장하세요:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: apache.192.168.164.131.sslip.io   # ← YOUR-IP에 위에서 복사한 IP 넣기!
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

### ➕ 적용:
```bash
kubectl apply -f apache-ingress.yaml
```

### 🌐 5단계: 웹 접속해 보기!
* 웹 브라우저 주소창에 아래 주소 입력:

```bash
curl http://apache.192.168.164.131.sslip.io
 🚀 Apache 웹서버의 테스트 페이지가 보이면 성공!
```

### 🎉 보너스: HTTPS 사용하기 (TLS 인증서)
* 만약 HTTPS를 사용하고 싶다면 self-signed나 cert-manager를 추가로 설치해야 합니다.
* (이 부분은 고급 옵션이므로 이후에 진행해도 됩니다.)

### ✅ 정리 – 순서 요약
| 순서 | 내용         | 명령                                |
| -- | ---------- | --------------------------------- |
| 1  | Ingress 설치 | `helm install ...`                |
| 2  | Apache 배포  | `kubectl create deployment ...`   |
| 3  | 서비스 만들기    | `kubectl expose ...`              |
| 4  | Ingress 정의 | YAML 작성 & `kubectl apply`         |
| 5  | 접속 테스트     | 웹에서 `http://apache.<IP>.sslip.io` |


## B. Ingress 설정 - https 접속(selfsigned)

### 💡 1. HTTPS가 뭐예요?
* HTTP는 인터넷 주소창에 http://로 시작하는 거예요.
* HTTPS는 s가 붙은 "안전한 버전"이에요.
* 누군가 내 웹사이트 내용을 몰래 보거나 바꾸지 못하도록 보호해줘요.

### 🧙 2. cert-manager는 누구인가요?
* 📜 cert-manager는 "마법사"예요.
* 내가 HTTPS 인증서가 필요하다고 말하면 알아서 만들어줘요.

#### ✅ 목표
* 웹사이트에 HTTPS 인증서 설치
* 주소창에 🔒 자물쇠 아이콘 뜨게 만들기

### 📦 3. cert-manager 설치하기 (마법사 부르기)
```bash
kubectl apply --validate=false -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml

kubectl get pods -n cert-manager
# 모두 Running 상태가 되면 OK!
```

###  4. 인증서 만드는 발급자(Issuer) 만들기
* 아래 YAML 파일을 저장해요 (이름: selfsigned-issuer.yaml):

```bash
vi selfsigned-issuer.yaml
```
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
```

```bash
kubectl apply -f selfsigned-issuer.yaml
kubectl get clusterissuer
# 🪄 이제 인증서를 "직접 만들어주는 발급자"가 생겼어요!
```

### 📃 5. HTTPS용 인증서 만들기
* 아래 YAML을 apache-cert.yaml로 저장:
```bash
vi apache-cert.yaml
```
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: apache-cert
spec:
  secretName: apache-tls              # 이 Secret이 자동 생성돼요!
  duration: 8760h                     # 1년 (선택사항)
  renewBefore: 360h                   # 만료 15일 전에 갱신
  subject:
    organizations:
    - MyOrg
  dnsNames:
  -  apache.192.168.164.131.sslip.io  # 도메인 이름
  issuerRef:
    name: selfsigned-issuer           # 위에서 만든 발급자 사용
    kind: ClusterIssuer
```
```bash
kubectl apply -f apache-cert.yaml
kubectl get certificate
kubectl get secret apache-tls
# 🗝️ apache-tls라는 이름의 인증서가 생기면 OK!
```

### 🧱 6. Ingress 수정해서 HTTPS 적용하기
* 이제 웹사이트에 🔒 자물쇠를 다는 거예요!
* 아래 YAML (apache-ingress-tls.yaml) 저장:

```bash
vi apache-ingress-tls.yaml
``` 
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: selfsigned-issuer
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
```

```bash
kubectl apply -f apache-ingress-tls.yaml
```

### 🌐 7. 접속 방법
```bash
curl https://apache.192.168.164.131.sslip.io
# 🔐 자물쇠 뜨면 성공! (혹시 경고가 떠도 괜찮아요, self-signed는 본인용이니까) 
```


## C. Ingress 설정 - Let's Encrypt로 진짜 인증서 발급하는 방법
(✅ 무료이고, 🔒 브라우저에서 자물쇠 표시가 뜹니다!)

### 🔐 목표
* 내 쿠버네티스 클러스터에 다음을 설정해서 진짜 HTTPS 인증서 사용하기:
  1. cert-manager 설치
  2. Let's Encrypt 발급자(ClusterIssuer) 설정 : ClusterIssuer 만들기 (Let's Encrypt ACME)
  3. Ingress 설정 - Ingress에 적용해서 HTTPS 접속

### ✅ 준비 사항
항목	설명
| 항목                | 설명                                    |
| ----------------- | ------------------------------------- |
| cert-manager 설치됨  | 👉 이미 설치했다면 OK                        |
| nginx ingress 설치됨 | 👉 이미 `helm`으로 설치했다면 OK               |
| 공인 IP or 도메인 있음   | 🟡 예: `apache.192.168.164.131.sslip.io`  |
| 클러스터 외부에서 접속 가능   | Let's Encrypt가 인증을 하려면 외부에서 접근 가능해야 함 |

### 🧙 1단계: ClusterIssuer 만들기 (Let's Encrypt ACME)
```bash
vi letsencrypt-issuer.yaml # 파일 생성:
```
```yaml
apiVersion: cert-manager.io/v1                  # ✅ 필수: 올바른 apiVersion
kind: ClusterIssuer                             # Cluster 전체에서 사용 가능한 인증 발급자
metadata:
  name: letsencrypt-http                        # 발급자 이름 (Ingress에서 참조할 이름)
spec:
  acme:
    email: iwbaek@gmail.com                     # 📧 알림 받을 이메일 주소
    server: https://acme-v02.api.letsencrypt.org/directory  # Let's Encrypt 프로덕션 서버
    privateKeySecretRef:
      name: letsencrypt-http-private-key        # 인증키를 저장할 Secret 이름
    solvers:
      - http01:
          ingress:
            class: nginx                        # 사용 중인 Ingress Controller 클래스

```
```bash
kubectl apply -f letsencrypt-issuer.yaml
``` 

### 🔍 2단계: 발급자 확인
```bash
kubectl get clusterissuer
# NAME                READY   AGE
# letsencrypt-http    True    10s
# ✅ READY: True면 성공입니다.
```


### 📜 3단계: HTTPS 인증서 발급 Ingress 설정
* 아래는 Ingress 설정 예 (apache-ingress-le.yaml):

```bash
vi apache-ingress-le.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apache-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-http  # <-- ClusterIssuer 이름 지정
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - apache.192.168.164.131.sslip.io
      secretName: apache-tls                          # 인증서가 저장될 Secret
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
```

### 🔧 적용:

```bash
kubectl apply -f apache-ingress-le.yaml
```

### ⏳ 4단계: 인증서 발급 상태 확인
```bash
kubectl get certificate
# NAME          READY   SECRET        AGE
# apache-tls    True    apache-tls    1m
# ✅ READY: True면 인증서 발급 완료!
```

### 🌐 5단계: 웹 브라우저에서 접속해 보기
```bash
curl https://apache.192.168.164.131.sslip.io
# 🔒 자물쇠 아이콘이 뜨고, 보안 연결됨이라고 표시되면 성공입니다!
```

### 🛠️ 문제 해결 팁
| 문제                       | 해결 방법                                                |
| ------------------------ | ---------------------------------------------------- |
| 인증서 발급 안 됨               | `kubectl describe certificate apache-tls` 확인         |
| ClusterIssuer 상태 `False` | `kubectl describe clusterissuer letsencrypt-http` 확인 |
| 인증 실패 메시지                | cert-manager logs 보기                                 |
| "DNS 문제"                 | 도메인이 외부에서 접근 가능해야 합니다                                |
| 외부 IP 없음                 | LoadBalancer 대신 `ngrok`, `nip.io`, `sslip.io` 활용 가능  |


### ✅ 요약 순서
1. cert-manager 설치
2. ClusterIssuer (Let's Encrypt) 만들기
3. Ingress 설정에서 TLS + cert-manager.io/cluster-issuer 설정
4. Secret과 인증서 자동 생성 확인
5. 웹 접속: https://apache.192.168.164.131.sslip.io

## D. 초기 상태에서 Let's Encrypt 기반 TLS Ingress 환경을 일괄 설정하는 전체 스크립트