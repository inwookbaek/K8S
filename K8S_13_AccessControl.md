# 13. 접근 제어
* **접근 제어(Access Control)**는 누가 어떤 리소스에 어떤 작업을 할 수 있는지를 제어하는 메커니즘입니다.
* 주로 RBAC(Role-Based Access Control) 방식을 사용

## 🔐 Kubernetes 접근 제어 구성 요소 요약
| 구성 요소                                | 설명                                 |
| ------------------------------------ | ---------------------------------- |
| **사용자(User)**                     | 클러스터 외부 또는 내부 인증된 주체               |
| **서비스 계정(ServiceAccount)**      | 파드나 앱이 클러스터 내부에서 API 접근할 때 사용하는 ID |
| **Role / ClusterRole**               | 리소스에 대한 권한 정의 (읽기/쓰기 등)            |
| **RoleBinding / ClusterRoleBinding** | 사용자/서비스계정에 Role을 부여                |

### ✅ 1. 인증(Authentication)
* 사용자를 식별 (예: kubeconfig, OIDC, TLS 인증서 등)
* 인증만으로는 아무 것도 할 수 없습니다 — 권한이 필요

### ✅ 2. 인가(Authorization)
* RBAC을 통해 특정 리소스에 대한 허용/거부를 판단
* kubectl 명령을 통해 확인 가능:

```bash
kubectl auth can-i get pods --as=user1
```

### ✅ 3. RBAC 구성 예시
#### 📄 (1) Role 예제 (네임스페이스 안에서만 권한 부여)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
#### 📄 (2) RoleBinding 예제 (Role을 사용자에게 연결)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 🌍 ClusterRole / ClusterRoleBinding (전체 클러스터에 적용)
* Role/RoleBinding은 네임스페이스 한정
* ClusterRole/ClusterRoleBinding은 전체 클러스터에서 작동

### 🔍 유용한 명령어
```bash
kubectl auth can-i delete pods --as=alice -n dev
# 권한이 있으면 yes, 없으면 no 반환
```

### 🚫 기본적으로 Kubernetes는 접근을 허용하지 않음!
* 아무 Role이 부여되지 않으면, 모든 요청은 거부됨
* 반드시 필요한 최소 권한만 부여하는 **최소 권한 원칙(Principle of Least Privilege)**을 따르는 것이 중요

## 13.1 사용자 인증 (Authentication)
* **사용자 인증(Authentication)**은 클러스터에 접근하려는 주체(사용자, 서비스 등)가 누구인지 식별하는 과정입니다.
* 즉, “이 요청을 보낸 사람이 누구인가?” 를 확인하는 단계

### 🔐 인증(Authentication) vs 인가(Authorization)
| 구분                 | 설명                     |
| ------------------ | ---------------------- |
| 인증(Authentication) | "누구인가?" 확인 → **식별**    |
| 인가(Authorization)  | "할 수 있는가?" 확인 → **권한** |

### ✅ Kubernetes에서 지원하는 인증 방법
| 인증 방법                     | 설명                                        |
| --------------------------- | ----------------------------------------- |
| **X.509 클라이언트 인증서** | 인증서 기반 인증 (주로 admin, kubeconfig에서 사용)     |
| **Static Token File**       | 토큰 기반 인증 (테스트 환경에서 사용됨)                   |
| **ServiceAccount Token**    | 파드 내부에서 API 서버 접근 시 자동 사용됨                |
| **OpenID Connect (OIDC)**   | 기업 SSO(Google, AzureAD, Keycloak 등) 연동 가능 |
| **Webhook Token Auth**      | 외부 시스템에 인증 위임 (커스텀 인증 서버)                 |
| **Bootstrap Token**         | 클러스터 초기 부트스트랩용 인증                         |
| **Proxy 인증**              | Proxy 서버를 통한 대리 인증 |

### 📌 사용자 vs ServiceAccount
| 항목    | 사용자(User)       | 서비스 계정(ServiceAccount) |
| ----- | --------------- | ---------------------- |
| 목적    | 사람이 API 요청      | 파드나 앱이 API 요청          |
| 관리 위치 | kubeconfig 등 외부 | 네임스페이스 내부 자동 생성        |
| 인증 방식 | 인증서, 토큰, OIDC   | Secret 토큰              |


### 13.1.1 HTTP Basic Authentication

#### HTTP Basic Authentication 은 deprecated 되어서 테스트 skip 할 것 ("13.1.2 X.509 인증서"로 스킴)

* Ingress리소스부분에서 살펴본 HTTP  Basic Auth인증과 마찬가지로 쿠버네티스 마스터에서 Basic Auth를 사용할 수 있다.
* HTTP Basic Authentication은 Kubernetes API 서버에서 간단한 사용자 인증을 위해 ID와 비밀번호를 Base64로 인코딩해서 요청 헤더에 포함하는 방식입니다.
* 초기 Kubernetes 버전에서 사용되었지만, 현재는 비권장(deprecated) 방식이며 프로덕션에서는 사용을 피해야 합니다.

#### ⚠️ 현재 상태: 기본 인증은 더 이상 권장되지 않음
* Kubernetes 1.19부터 --basic-auth-file 옵션은 deprecated
* Kubernetes 1.25에서 완전히 제거됨
* 보안이 취약하고, 사용자 관리가 어렵기 때문에 X.509 인증서 또는 OIDC 연동을 사용해야 합니다

### 🔐 인증서 or OIDC
| 인증 방법              | 보안성   | 관리 편의성  | 추천 용도       |
| ------------------ | ----- | ------- | ----------- |
| **X.509 인증서**      | 매우 높음 | 복잡함     | 관리자, CI/CD  |
| **OIDC (OAuth2)**  | 매우 높음 | 좋음      | SSO, 사용자 관리 |
| **ServiceAccount** | 높음    | 자동화에 적합 | 앱, 파드 내부 요청 |

### 🚫 왜 Basic Auth를 쓰지 말아야 하나요?
* 평문 비밀번호를 저장해야 함
* 암호 변경 어려움
* RBAC과 연동 불편
* 토큰/인증서보다 훨씬 취약

### 📌 결론
* HTTP Basic Authentication은 더 이상 사용하지 마세요.
* 학습이나 데모용이 아니라면 X.509 인증서, ServiceAccount, OIDC 인증 방식을 사용하세요.


```bash
# 만약에 $HOME/.kube/config파일을 잘못 수정했을 경우에 아래 명령 싫행
mkdir -p $HOME/.kube
sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config # ~/.kube/config 파일의 소유자를 현재 로그인한 사용자로 변경하는 명령


cat $HOME/.kube/config
# apiVersion: v1                    # 이 설정 파일의 API 버전 (항상 v1)
# kind: Config                      # 이 파일은 Kubernetes 클러스터 접속 정보인 "Config" 타입임
# preferences: {}                  # 사용자 환경 설정 (일반적으로 비워둠)

# clusters:                        # 연결 가능한 Kubernetes 클러스터 목록
# - cluster:
#     certificate-authority-data: LS0tLS1CRUdJTiB...g==  # 클러스터의 인증서 (Base64 인코딩된 CA 인증서)
#     server: server: https://127.0.0.1:6443                     # API 서버 주소
#   name: default                                         # 위 클러스터 설정의 이름

# contexts:                        # 어떤 사용자와 어떤 클러스터를 함께 쓸지 지정하는 '컨텍스트' 목록
# - context:
#     cluster: default             # 사용할 클러스터 이름
#     user: default                # 사용할 사용자 이름
#   name: default                  # 이 컨텍스트의 이름

# current-context: default         # 현재 활성화된 컨텍스트 (kubectl이 사용할 기본 컨텍스트)

# users:                           # 클러스터에 접근할 수 있는 사용자 계정 목록
# - name: default
#   user:
#     username: admin              # 사용자 이름 (기본 인증 방식)
#     password: 7e92dba7..         # 사용자 비밀번호
```
### 📌 전체 구성 요약
* clusters → 접속할 Kubernetes 서버 정보
* users → 접속할 사용자 정보 (인증 수단)
* contexts → cluster + user 조합
* current-context → kubectl이 기본으로 사용할 컨텍스트

### ⚠️ 보안 주의
* 위 예시는 HTTP Basic Authentication을 사용합니다. (username/password)
* 최신 Kubernetes에서는 이 방식이 더 이상 권장되지 않으며, 인증서 또는 토큰 기반 인증 사용을 권장합니다.
* 비밀번호는 평문 또는 kubeconfig 파일에 저장되므로 보안상 취약합니다.


```bash
# 헤더 없이 접속
# username, password정보를 가지고 master에 접속
# -k	서버의 SSL 인증서를 검증하지 않음 (자체 서명된 인증서에도 연결 가능)
# -v	자세한 디버깅 정보 출력 (요청/응답 헤더, 연결 정보 등 포함)
curl -kv https://127.0.0.1:6443/api
# HTTP/1.1 401 Unauthorized
# Www-Authenticate: Basic realm="kubernetes-master"
# {
#   "kind": "Status",
#   "apiVersion": "v1",
#   "metadata": {
#   },
#   "status": "Failure",
#   "message": "Unauthorized",
#   "reason": "Unauthorized",
#   "code": 401
# }

echo -n myuser:mypassword | base64
# basic auth 설정
curl -kv -H "Authorization: Basic $(echo -n admin:7e92dba7.. | base64)" https://127.0.0.1:6443/api
curl -kv -H "Authorization: Basic $(echo -n myuser:mypassword | base64)" https://127.0.0.1:6443/api

# HTTP/1.1 200 OK
# {
#   "kind": "APIVersions",
#   "versions": [
#     "v1"
#   ],
#   "serverAddressByClientCIDRs": [
#     {
#       "clientCIDR": "0.0.0.0/0",
#       "serverAddress": "172.31.17.32:6443"
#     }
#   ]
# }
```

```bash
# -v=7	verbose 로그 레벨을 7로 설정 (API 요청과 응답 세부 정보 출력)
kubectl get pod -v 7
# I0530 ...] Config loaded from file: /home/ubuntu/.kube/config
# I0530 ...] GET https://127.0.0.1:6443/.../default/pods?limit=500
# I0530 ...] Request Headers:
# I0530 ...]     Accept: application/json;as=Table;v=v1;g=...
# I0530 ...]     Authorization: Basic <masked>
```

```bash
# 사용자추가 : 마지막행에 "password,user,uid,group1[,group2,group3]" 추가
# sudo로 작업
sudo vi /var/lib/rancher/k3s/server/cred/passwd
```

```bash
# /var/lib/rancher/k3s/server/cred/passwd 

# c8ae61726384c19726022879dea9dd66,node,node,k3s:agent
# c8ae61726384c19726022879dea9dd66,server,server,k3s:server
# fcb41891d94b1a362cf7ccc4086c2465,admin,admin,system:masters
# mypassword,myuser,myuser,system:masters
```

```
# 형식은 다음과 같습니다.
password,user,uid,group1[,group2,group3]
```

```yaml
# vi $HOME/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiB...g==
    server: https://192.168.164.130:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: mypassword   # 비밀번호 수정
    username: myuser       # 계정이름 수정
```

```bash
# 실험용 nginx Pod 생성
kubectl run nginx --image nginx
# error: You must be logged in to the server (Unauthorized)
# error: error loading config file "/home/gilbaek/.kube/config": illegal base64 data at input byte 15
```

```bash
sudo systemctl restart k3s.service

# 정상적으로 Pod가 생성됩니다.
kubectl run nginx --image nginx
# pod/nginx created

# 조회도 가능합니다.
kubectl get pod
# NAME      READY   STATUS    RESTARTS   AGE
# nginx     1/1     Running   0          5s

# Pod 삭제도 가능합니다.
kubectl delete pod nginx
# pod/nginx deleted
```

### 13.1.2 X.509 인증서
* HTTPS통신을 하기 위해 인증서(X.509)를 서버측에 제공
* 인증서는 쿠버네티스가 인증한 사용자를 사용해야 하는데 이를 위해 CSR(Certificate Signing Request)를 사용하여 CA로부터 서명을 받아야 한다.
* TLS 기반의 안전한 통신과 사용자 식별을 제공합니다.
* 관리자와 자동화된 시스템 간 안전한 API 접근에 자주 사용됩니다.

#### 📦 필요한 구성 요소
| 파일명          | 설명                                          |
| ------------ | ------------------------------------------- |
| `myuser.key` | 사용자의 **개인 키** (private key)                 |
| `myuser.crt` | 사용자의 **공개 인증서** (client certificate)        |
| `ca.crt`     | Kubernetes API 서버가 사용하는 **CA 인증서** (서명 확인용) |

###### 사용자인증을 만들기 위해 cloudflare의 cfssl툴을 설치
🔍 명령어 설명
| 옵션                | 설명                              |
| ----------------- | ------------------------------- |
| `-q`              | 조용하게 실행 (진행 상황 외에는 출력 안 함)      |
| `--show-progress` | 다운로드 진행률 출력                     |
| `--https-only`    | HTTPS만 허용 (보안)                  |
| `--timestamping`  | 이미 있는 파일이면 수정 시간 비교해서 재다운 여부 결정 |

```bash
wget -q --show-progress -O cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 && \
wget -q --show-progress -O cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

chmod +x cfssl cfssljson 
sudo mv cfssl cfssljson /usr/local/bin/
# 설치확인
cfssl version
cfssljson --help
```

##### 인증서생성하는 방법
1. CSR파일 생성
2. CA로부터 인증서 서명
3. 발급된 인증서(인증서 및 개인키)를 KUBECONFIG파일에 설정


```bash
# 사용자 CSR 파일 생성
cat > client-cert-csr.json <<EOF
{
  "CN": "client-cert",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "O": "system:masters"
    }
  ]
}
EOF
```
##### 🔍 설명:
* CN: Common Name (사용자 이름, 예: client-cert)
* O: 조직, Kubernetes에서는 system:masters는 클러스터 관리자 권한 부여

##### 생성한 CSR파일을 이용하여 쿠버네티스에 CA서명을 요청
* k3s 쿠버네티스 CA는 다음 위치에 존재한다.
  - 인증서: `/var/lib/rancher/k3s/server/tls/client-ca.crt`
  - 개인키: `/var/lib/rancher/k3s/server/tls/client-ca.key`

```bash 
sudo cat /var/lib/rancher/k3s/server/tls/client-ca.crt
sudo cat /var/lib/rancher/k3s/server/tls/client-ca.key

# 사용자 인증서를 발급하기 위한 CA Config파일을 생성
cat > rootCA-config.json <<EOF
{
  "signing": {
    "profiles": {
      "root-ca": {
        "usages": ["signing", "key encipherment", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

# 사용자인증서를 발급
sudo cfssl gencert \
  -ca=/var/lib/rancher/k3s/server/tls/client-ca.crt \
  -ca-key=/var/lib/rancher/k3s/server/tls/client-ca.key \
  -config=rootCA-config.json \
  -profile=root-ca \
  client-cert-csr.json | cfssljson -bare client-cert

ls -al
# client-cert-csr.json
# client-cert-key.pem
# client-cert.csr
# client-cert.pem
# rootCA-config.json

# 중요한 문서는 아래 pem파일이며 CSR을 통해 CA로부터 사용자 인증서와 개인키를 발급 받은 것
# client-cert.pem
# client-cert-key.pem
```

```bash
# 인증서와 개인키를 ~/.kube/config 파일에 설정
# Kubernetes kubeconfig 파일(~/.kube/config)에 새 사용자(x509)를 등록
kubectl config set-credentials x509 \
          --client-certificate=client-cert.pem \
          --client-key=client-cert-key.pem \
          --embed-certs=true

# User "x509" set.

cat .kube/config
# ...
# - name: x509
#   user:
#     client-certificate-data: LS0tLS1CRUdJTiB...
#     client-key-data: LS0tLS1CRUd...
# ...
```
##### 🧩 옵션 설명:
| 옵션                                     | 설명                                                                                               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `set-credentials x509`                 | 사용자 이름을 `x509`로 설정합니다. 이 이름은 나중에 context에 연결할 수 있습니다.                                            |
| `--client-certificate=client-cert.pem` | 클라이언트 인증서 (`cfssl`로 생성한 `client-cert.pem`) 경로입니다.                                                |
| `--client-key=client-cert-key.pem`     | 인증서에 해당하는 개인 키 경로입니다. (`client-cert-key.pem`)                                                    |
| `--embed-certs=true`                   | 인증서와 키 내용을 `.kube/config`에 **직접 포함(embed)** 시켜, 파일 없이도 작동하게 합니다. 인증서가 삭제되어도 `kubectl`은 계속 작동합니다. |


```bash
kubectl config set-context default --user=x509
# Context "default" modified.
# - context:
#     cluster: default
#     user: default ---> x509 로 변경
```
##### 🔍 kubectl config set-context 명령의 목적:
* Kubernetes의 kubeconfig 설정에서 default라는 context에 x509 사용자를 연결(또는 교체) 하기 위한 명령입니다.
* 즉: 기존에 정의된 default context가 있다면 그 안의 사용자 정보를 x509로 변경하고,
* 없다면 새로운 default context를 생성하면서 x509 사용자를 설정합니다.

##### 🧩 주요 요소 설명:
| 항목            | 설명                                                                                  |
| ------------- | ----------------------------------------------------------------------------------- |
| `set-context` | `kubeconfig`에 context를 설정(생성/수정)하는 명령입니다.                                           |
| `default`     | context 이름입니다. 사용자 정의 이름도 가능하지만, 여기선 기본 context 이름인 `default`를 사용하고 있습니다.           |
| `--user=x509` | 해당 context에서 사용할 사용자 이름입니다. 앞서 설정한 `kubectl config set-credentials x509` 명령과 연계됩니다. |

```bash
# client-certificate과 key가 base64로 인코딩되어 삽입되어 있습니다.
cat $HOME/.kube/config
# apiVersion: v1
# clusters:
# - cluster:
#     certificate-authority-data: LS0tLS1CRUdJTiB...g==
#     server: https://127.0.0.1:6443
#   name: default
# contexts:
# - context:
#     cluster: default
#     user: x509
#   name: default
# current-context: default
# kind: Config
# preferences: {}
# users:
# - name: default
#   user:
#     password: mypassword
#     username: myuser
# - name: x509
#   user:
#     client-certificate-data: LS0tLS1CRUdJTiB...
#     client-key-data: LS0tLS1CRUdJTiBSU0EgUFJ...
```

```bash
# x509 인증방식으로 변경 이제 pod을 생성
# 직접 생성한 x509 인증서를 이용하여 사용자 인증을 받아 보았다.
# 많은 쿠버네티스배포판에서 내부 컴퍼넌트끼리 서로 인증할 때 x509를 이용한다.
kubectl run nginx --image nginx
# pod/nginx created

# 조회
kubectl get pod
# NAME     READY   STATUS    RESTARTS   AGE
# nginx    1/1     Running   0          5s

# 삭제
kubectl delete pod nginx
# pod/nginx deleted
```

## 13.2 역할 기반 접근 제어 (RBAC)
* 사용자 인증 후 권하허가 단계에서는 역할기반접근제어를 통해 사용자들의 권하늘 관리한다.
* RBAC는 사용자나 그룹의 역할을 기반으로 쿠버네티스의 다양한 리소스의 접근을 관리하는 방법을 말한다.

### 🔐 1. RBAC이란?
* RBAC는 Kubernetes 클러스터 내 리소스에 대해 "누가", "무엇을", "어디서" 할 수 있는지를 정의합니다.
* 예시로 정리하면:
  - “사용자 alice는 default 네임스페이스에서 pods를 조회(get)할 수 있다.”
  - “서비스 계정 ci-bot은 deployments를 생성하고 수정할 수 있다.”

### 🧩 2. RBAC 구성 요소
* Kubernetes RBAC은 다음 네 가지 리소스 객체로 구성됩니다:
| 리소스                  | 설명                                                |
| -------------------- | ------------------------------------------------- |
| `Role`               | **네임스페이스 범위의 권한**을 정의.                            |
| `ClusterRole`        | **클러스터 전체 또는 모든 네임스페이스에 대한 권한**을 정의.              |
| `RoleBinding`        | 특정 `Role`을 사용자나 서비스 계정에 **바인딩(연결)**.              |
| `ClusterRoleBinding` | `ClusterRole`을 클러스터 또는 모든 네임스페이스에서 사용자에게 **바인딩**. |

### 🔍 3. 자주 쓰는 verbs (동작 권한)
| Verb     | 설명    |
| -------- | ----- |
| `get`    | 객체 조회 |
| `list`   | 목록 조회 |
| `watch`  | 변경 감시 |
| `create` | 생성    |
| `update` | 수정    |
| `patch`  | 일부 수정 |
| `delete` | 삭제    |

### ✅ 4. 요약
* RBAC는 인증된 사용자가 Kubernetes 리소스에 대해 무엇을 할 수 있는지 제어합니다:
  - Role/ClusterRole: 권한 정의
  - RoleBinding/ClusterRoleBinding: 사용자와 권한 연결

### 13.2.1 Role (ClusterRole)
#### Role vs ClusterRole
| 구분        | Role                              | ClusterRole                                                |
| --------- | --------------------------------- | ---------------------------------------------------------- |
| 범위        | **네임스페이스(namespace) 단위** 권한 부여    | **클러스터 전체(cluster-wide)** 권한 부여                            |
| 사용처       | 특정 네임스페이스 내에서만 적용                 | 모든 네임스페이스 또는 클러스터 리소스에 적용 가능                               |
| 권한 대상 리소스 | 네임스페이스 리소스 (예: pods, services)    | 네임스페이스 리소스 또는 클러스터 리소스 (예: nodes, persistentvolumes) 모두 가능 |
| 바인딩 대상    | `RoleBinding`을 통해 사용자나 서비스 계정에 연결 | `RoleBinding` 또는 `ClusterRoleBinding`으로 연결 가능              |
| 예시        | `default` 네임스페이스에서 pod 조회 권한      | 클러스터 전체에서 노드 조회 권한, 네임스페이스에 구애받지 않는 권한                     |


```yaml
# vi K8S/CH05/role.yaml
apiVersion: rbac.authorization.k8s.io/v1  # Role 객체를 정의하는 최신 API 그룹
kind: Role                                 # 네임스페이스 단위 권한을 정의하는 Role
metadata:
  name: pod-viewer                        # Role의 이름
  namespace: default                      # 이 Role이 속한 네임스페이스 (여기선 default)

rules:
- apiGroups: [""]                         # ""은 core API group을 의미 (예: pods, services 등)
  resources:                              # 접근을 허용할 리소스 종류
  - pods                                  # pods 리소스에 대해
  verbs:                                  # 허용할 동작(권한)을 정의
  - get                                   # 개별 pod 정보 조회
  - watch                                 # pod 변경 사항 실시간 감시
  - list                                  # pod 목록 조회
```
###### ✅ 요약:
* 이 Role은 default 네임스페이스 안에서 pods 리소스에 대해 get, watch, list 권한을 부여합니다. 
* 이 Role을 실제 사용자나 서비스 계정에 적용하려면 별도로 RoleBinding을 생성해야 합니다.

```bash
kubectl apply -f K8S/CH05/role.yaml
# role.rbac.authorization.k8s.io/pod-viewer created

kubectl get role
# NAME         CREATED AT
# pod-viewer   2020-05-31T17:16:47Z
```

### 13.2.2 Subjects
* subjects는 Kubernetes RBAC에서 RoleBinding 또는 ClusterRoleBinding 리소스를 통해 권한을 부여할 대상(사람, 사용자, 그룹, 서비스 계정 등)을 지정하는 항목입니다.
* 쿠버네티스에는 User와 Group이라는 리소스가  명시적으로 구현되어 있지는 않고 개념적으로만 존재한다.
* X509에서는 인증서의 CN이 user, O(Oraganization)은 Group로 인식된다.

#### 📌 ServiceAccount?
* ServiceAccount는 Kubernetes 클러스터 내부에서 실행 중인 Pod가 API 서버와 통신할 때 사용되는 계정입니다. 
* 사용자 계정(User)과는 달리, 자동으로 생성되고 Pod에 연결되어 인증/인가를 수행하는 **비인간 사용자(머신 계정)**

##### 📌 ServiceAccount 개념 요약 
| 항목    | 설명                                                     |
| ----- | ------------------------------------------------------ |
| 목적    | Pod가 Kubernetes API에 접근할 수 있도록 인증 정보 제공                |
| 기본 계정 | `default` 네임스페이스에 `default`라는 이름의 ServiceAccount 자동 생성 |
| 인증 방법 | Token (JWT) 방식으로 API 서버에 인증                            |
| 연결 방법 | Pod의 `spec.serviceAccountName` 필드로 지정                  |

```bash
kubectl get serviceaccount  # 또는 sa
# NAME      SECRETS   AGE
# default   1         28h

kubectl get serviceaccount default -oyaml
# apiVersion: v1
# kind: ServiceAccount
# metadata:
#   creationTimestamp: "2020-06-07T10:08:29Z"
#   name: default
#   namespace: default
#   resourceVersion: "292"
#   selfLink: /api/v1/namespaces/default/serviceaccounts/default
#   uid: 0183509b-2e36-412d-b229-048f09b2afc1
# secrets:
# - name: default-token-vkrsk
```

#### ServiceAccount 생성
* ServiceAccount리소스는 네임스페이스 레벨에서 동작하며, 사용자가 Pod리소스 생성시 , 명시적으로 지정하지 않으면 default ServiceAccount가 사용된다.
* 따라서, 모든 namespace를 생성할 때 에는 default ServiceAccount가 자동으로 생성된다.
* ServiceAccount의 목적은 사용자가 아닌 Pod이 쿠버네티스와 통신할 때 사용하는 신원(Identity)이다.

```bash
kubectl create sa mysa
# serviceaccount/mysa created

# sa = serviceaccount
kubectl get sa 
# NAME      SECRETS   AGE
# default   1         28h
# mysa      1         10s
```

### 13.2.3 RoleBinding (ClusterRoleBinding)
* RoleBinding리소스는 Role과 Subjects를 엮는 역할을 담당하여 특정 사용자가 부여받은 특정 권하늘 사용할 수 있게 한다. 
* 즉, RoleBinding과 ClusterRoleBinding은 Kubernetes RBAC(Role-Based Access Control) 에서 권한을 사용자/그룹/ServiceAccount에 연결해주는 리소스

#### 🔑 기본 개념 차이
| 항목       | RoleBinding           | ClusterRoleBinding        |
| -------- | --------------------- | ------------------------- |
| 적용 범위    | **네임스페이스 단위**         | **클러스터 전체**               |
| 대상 권한    | Role 또는 ClusterRole   | ClusterRole만 가능           |
| 일반 사용 용도 | 네임스페이스 내 리소스 접근 권한 부여 | 모든 네임스페이스나 클러스터 리소스 접근 권한 |


```yaml
# vi K8S/CH05/read-pods.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods            # RoleBinding의 이름
  namespace: default         # 이 RoleBinding이 적용될 네임스페이스

subjects:
- kind: ServiceAccount       # 권한을 부여할 주체의 종류 (여기서는 ServiceAccount)
  name: mysa                 # 대상 ServiceAccount의 이름
  namespace: default         # ServiceAccount가 속한 네임스페이스 (명시 권장)

roleRef:
  kind: Role                 # 참조할 권한 리소스의 종류 (Role 또는 ClusterRole)
  name: pod-viewer           # 부여할 Role의 이름
  apiGroup: rbac.authorization.k8s.io  # Role이 속한 API 그룹

```

```bash
kubectl apply -f K8S/CH05/read-pods.yaml
# rolebinding.rbac.authorization.k8s.io/read-pods created

kubectl get rolebinding
# NAME        ROLE              AGE
# read-pods   Role/pod-viewer   20s
```

#### pod를 생성하고 exec명령을 이용하여 내부로 접근하기
```yaml
# vi K8S/CH05/nginx-sa.yaml
apiVersion: v1               # Kubernetes core API 그룹의 버전
kind: Pod                    # 리소스 종류: Pod

metadata:
  name: nginx-sa             # 생성될 Pod의 이름

spec:
  serviceAccountName: mysa   # 이 Pod가 사용할 ServiceAccount 이름
  containers:
  - name: nginx              # 컨테이너 이름
    image: nginx             # 사용할 Docker 이미지 (공식 nginx 이미지)
```

```bash
kubectl apply -f K8S/CH05/nginx-sa.yam
# pod/nginx-sa created

kubectl get pod nginx-sa -oyaml | grep serviceAccountName
#   serviceAccountName: mysa

# 내부 접근
kubectl exec -it nginx-sa -- bash
```

```bash
# kubectl 설치
$root@nginx-sa:/# curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.3/bin/linux/amd64/kubectl \
                 && chmod +x ./kubectl \
                 && mv ./kubectl /usr/local/bin
```
##### 📌 라인별 설명
* curl -LO https://...
  - -L: 리다이렉션을 따라감 (URL이 이동된 경우 따라감)
  - -O: 원래 파일 이름으로 저장 (여기선 kubectl)
  - 이 URL은 구버전이며, 현재는 https://dl.k8s.io로 변경됨
* chmod +x ./kubectl : 다운로드한 kubectl 바이너리에 실행 권한 부여
* mv ./kubectl /usr/local/bin : 실행 파일을 시스템 전역 명령 경로(/usr/local/bin)로 이동시킴
* 이제 kubectl 명령을 어디서나 사용할 수 있게 됨

```bash
# Pod 리소스 조회
$root@nginx-sa:/# kubectl get pod
# NAME      READY   STATUS    RESTARTS   AGE
# nginx-sa  2/2     Running   0          2d21h

# Service 리소스 조회
# Pod는 정상적으로 조회가 되지만 Service리소스는 권한이 없기 때문에 에러발생
$root@nginx-sa:/# kubectl get svc
# Error from server (Forbidden): services is forbidden: User 
# "system:serviceaccount:default:mysa" cannot list resource "services"
#  in API group "" in the namespace "default"
```

```bash
# Pod를 빠져나갑니다.
$root@nginx-sa:/# exit

# 호스트 서버에서 다음 명령을 수행
# Service리소스도 조회할 수 있도록 수정
# K8S/CH05/role.yaml
# ...   - services   # services 리소스 추가 
cat << EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-viewer
rules:
- apiGroups: [""]
  resources: 
  - pods
  - services   # services 리소스 추가
  verbs: 
  - get
  - watch
  - list
EOF
# role.rbac.authorization.k8s.io/pod-viewer edited

# 다시 Pod 접속
kubectl exec -it nginx-sa -- bash

# Service 리소스 조회
$root@nginx-sa:/# kubectl get svc
# NAME           TYPE           CLUSTER-IP      ...   
# kubernetes     ClusterIP      10.43.0.1       ...

# Pod를 빠져나갑니다.
$root@nginx-sa:/# exit
```

### Clean up

```bash
kubectl delete pod nginx-sa
kubectl delete rolebinding read-pods
kubectl delete role pod-viewer
kubectl delete sa mysa
```

## 13.3 네트워크 접근 제어 (Network Policy)
* 네트워크 접근 제어는 Kubernetes 클러스터 내에서 Pod 간의 통신을 제어하는 데 사용되며, 
* 이를 위해 Kubernetes에서는 NetworkPolicy 리소스를 사용합니다. 
* 이는 기본적으로 허용된 모든 트래픽을 제한하고, 명시된 규칙에 따라 허용된 트래픽만 통과하도록 제어한다

### 🔐 Network Policy란?
* Kubernetes 리소스 중 하나로, Pod 간의 네트워크 트래픽을 제어함
* 주로 보안 강화를 위해 특정 Pod에 대한 접근을 제한할 때 사용
* 트래픽은 기본적으로 허용됨 → NetworkPolicy가 적용되면 명시적으로 허용된 트래픽만 허용

### 📦 기본 구조 예시
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-from-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx  # 대상 Pod 선택 (수신자)
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend  # 허용할 송신자 Pod 선택
```
#### 📘 주요 필드 설명
| 필드            | 설명                                             |
| ------------- | ---------------------------------------------- |
| `podSelector` | 이 정책이 **적용되는 대상 Pod**를 선택합니다                   |
| `policyTypes` | `Ingress`, `Egress` 또는 둘 다 명시 가능               |
| `ingress`     | **수신 트래픽 제어** (Pod로 들어오는 트래픽)                  |
| `egress`      | **송신 트래픽 제어** (Pod에서 나가는 트래픽)                  |
| `from`, `to`  | 트래픽 허용 조건 (Pod, Namespace, IP block 등으로 정의 가능) |

### 13.3.1 Network Policy 모듈 설치 - Canal
* K3S는 기본적으로 flannel을 네트워크 제공자로 사용하고 있다.
* flannel에 Network Policy를 설정하기 위해 Canal을 설치한다.
* Canal은 Calico와 Flannel을 결합한 하이브리드 CNI로, 간단한 설치와 정교한 네트워크 정책 제어를 동시에 제공한다.

```bash
# 📦 Canal 설치 (Calico + Flannel)
# 이 명령은 다음을 수행합니다:
# Canal 구성 리소스를 설치 (Deployment, DaemonSet 등)
# calico-node, flannel, calico-kube-controllers 등의 Pod 배포
# kube-system 네임스페이스에 Canal 구성 적용

kubectl apply -f https://docs.projectcalico.org/manifests/canal.yaml
# configmap/canal-config created
# ...
# serviceaccount/canal created
# deployment.apps/calico-kube-controllers created
# serviceaccount/calico-kube-controllers created
```

### 13.3.2 쿠버네티스 네트워크 기본 정책
* 쿠버네티스 네트워크 기본 정책은 다음과 같다.
1. 기본적으로 클러스터에 네트워크 정책이 하나도 설정되어 있지 않다.
1. 설정된게 없으면 네임스페이스의 모든 트래픽은 열려 있다.
1. 한개의 네트워크 정책이라도 설정되면 정책의 영향을 받는 Pod에 대해서 해당 네트워크 정책 이외의 나머지 트래픽은 전부 막힌다.

### 13.3.3 `NetworkPolicy` 문법
* NetworkPolicy는 네이스페이스 레벨에서 동작하며 라벨셀렉터를 이용하여 특정 Pod에 네트워크 정책을 적용한다.

#### 🧱 기본 구조
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: <정책 이름>
  namespace: <대상 네임스페이스>
spec:
  podSelector:       # 어떤 Pod에 적용할 것인지 지정
    matchLabels:
      <key>: <value>
  policyTypes:       # 적용할 트래픽 방향 (Ingress, Egress)
  # ingress: {}      # 모든 트래픽을 허용    
  ingress:           # 들어오는 트래픽 허용 규칙
    - from:          # 어떤 대상에게서 오는 트래픽을 허용할지
      ports:         # 어떤 포트를 허용할지
  # egress: {}       # 모든 트래픽을 허용    
  egress:            # 나가는 트래픽 허용 규칙
    - to:            # 목적지
      ports:         # 허용 포트
```
#### 🎯 주요 필드 설명
| 필드            | 설명                                                      |
| ------------- | ------------------------------------------------------- |
| `podSelector` | 이 정책이 적용될 Pod들을 라벨로 지정                                  |
| `policyTypes` | `Ingress`, `Egress`, 또는 둘 다 지정 가능                       |
| `ingress`     | 허용할 **들어오는** 트래픽의 조건 정의                                 |
| `egress`      | 허용할 **나가는** 트래픽의 조건 정의                                  |
| `from` / `to` | 통신 대상 지정: `podSelector`, `namespaceSelector`, `ipBlock` |
| `ports`       | 허용할 포트: `port`, `protocol` 사용 (기본은 TCP)                 |

#### 📌 팁
* podSelector 생략 불가 (빈 {}는 모든 Pod)
* from과 to는 복합 조건 (AND) 으로 처리
* NetworkPolicy는 허용 기반 → 명시되지 않은 트래픽은 차단

### 13.3.4 네트워크 구성

```bash
# 테스트 Pod 생성
kubectl run client --image nginx
# pod/client created 
```

#### Private Zone
* Private Zone은 외부에서 직접 접근할 수 없는, 내부 전용 DNS 영역 또는 네트워크 영역을 말한다.
* 일반적으로 내부 네트워크에서만 접근 가능한 DNS 도메인 영역입니다. 
* 이는 **외부 DNS (예: public DNS like google.com)**와는 달리, 내부 시스템이나 클러스터 내에서만 유효하다.

##### 🧭 어디서 사용되나요?
| 환경                         | 설명                                                              |
| -------------------------- | --------------------------------------------------------------- |
| **클라우드 (AWS, GCP, Azure)** | VPC 전용 DNS 영역. 외부에서는 이름을 해석하거나 접근 불가                            |
| **Kubernetes**             | 내부 DNS (`svc.cluster.local`, `pod.namespace.svc`) 등은 외부에서 접근 불가 |
| **보안 인프라**                 | 내부 API, 데이터베이스, 내부 마이크로서비스용 이름 지정 시 사용                          |

##### 🚫 Public Zone과 비교
| 구분       | Public Zone    | Private Zone         |
| -------- | -------------- | -------------------- |
| 접근 가능 범위 | 전 세계 인터넷       | 내부 네트워크 (VPC, 클러스터)  |
| 예시       | `example.com`  | `app.internal.local` |
| 보안성      | 낮음 (외부 노출)     | 높음 (내부 접근만 허용)       |
| 사용 목적    | 공개 웹사이트, API 등 | 내부 서비스, DB, 사내 API 등 |

```yaml
# 먼저 기본적으로 전체 인바운드 트래픽을 차단하여 외부 트래픽이 들어올 수 없게 private zone으로 생성
# vi K8S/CH05/deny-all.yaml
kind: NetworkPolicy                         # 리소스의 종류: NetworkPolicy
apiVersion: networking.k8s.io/v1            # 사용하는 API 버전
metadata:
  name: deny-all                            # 네트워크 정책의 이름
  namespace: default                        # 이 정책이 적용될 네임스페이스
spec:
  podSelector: {}                           # 이 네임스페이스 내 모든 파드 대상 (빈 셀렉터는 모든 파드 의미)
  ingress: []                               # 들어오는 트래픽 전부 차단 (허용 규칙이 없음 = Deny All)
```

```bash
kubectl apply -f K8S/CH05/deny-all.yaml
# networkpolicy.networking.k8s.io/deny-all created

kubectl get networkpolicy
# NAME       POD-SELECTOR   AGE
# deny-all   <none>         2s

kubectl get networkpolicy deny-all -oyaml
# apiVersion: networking.k8s.io/v1
# kind: NetworkPolicy
# metadata:
#   ...
# spec:
#   podSelector: {}
#   policyTypes:
#   - Ingress
```

#### Web pod 오픈

```yaml
# vi K8S/CH05/web-open.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-open           # 이 정책의 이름
  namespace: default       # 적용할 네임스페이스
spec:
  podSelector:
    matchLabels:
      run: web             # run=web 라벨이 붙은 Pod에 적용
  ingress:
  - from:
    - podSelector: {}      # 모든 Pod에서 접근 허용
    ports:
    - protocol: TCP
      port: 80             # TCP 80번 포트만 허용
```

```bash
kubectl apply -f K8S/CH05/web-open.yaml
# networkpolicy.networking.k8s.io/deny-all created

# run=web 이라는 라벨을 가진 웹 서버 생성
kubectl run web --image nginx
# pod/web created 

# run=non-web 이라는 라벨을 가진 웹 서버 생성
kubectl run non-web --image nginx
# pod/non-web created 

# Pod IP 확인
kubectl get pod -owide
# NAME      READY   STATUS    RESTARTS   AGE     IP            NODE 
# web       1/1     Running   0          34s     10.42.0.169   master
# non-web   1/1     Running   0          32s     10.42.0.170   master
# client    1/1     Running   0          28s     10.42.0.171   master
```

```bash
# client Pod 진입
kubectl exec -it client -- bash

# web Pod 호출
$root@client:/# curl 10.42.0.169
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...

# non-web Pod 호출
$root@client:/# curl 10.42.0.170
# curl: (7) Failed to connect to 10.42.0.170 port 80: Connection refused

$root@client:/# exit
```

#### Web과의 통신만 허용된 app

```yaml
# vi K8S/CH05/allow-from-web.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-web     # 정책 이름
  namespace: default        # 적용할 네임스페이스
spec:
  podSelector:
    matchLabels:
      run: app              # 이 정책의 대상 Pod (run=app)
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: web          # run=web 라벨을 가진 Pod만 접근 가능
```

```bash
kubectl apply -f K8S/CH05/allow-from-web.yaml
# networkpolicy.networking.k8s.io/allow-from-web created

# run=app 이라는 라벨을 가진 앱 서버 생성
kubectl run app --image nginx
# pod/app created

# Pod IP 확인
kubectl get pod -owide
# NAME      READY   STATUS    RESTARTS   AGE     IP            NODE
# web       1/1     Running   0          34s     10.42.0.169   master
# non-web   1/1     Running   0          32s     10.42.0.170   master
# client    1/1     Running   0          28s     10.42.0.171   worker
# app       1/1     Running   0          28s     10.42.0.172   worker

# client Pod 진입
kubectl exec -it client -- bash

# client에서 app 서버 호출
$root@client:/# curl 10.42.0.172
# curl: (7) Failed to connect to 10.42.0.172 port 80: Connection refused

# client Pod 종료
$root@client:/# exit

# web Pod 진입
kubectl exec -it web -- bash

# web에서 app 서버 호출
$root@web:/# curl 10.42.0.172
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...

# web Pod 종료
$root@web:/# exit

# non-web Pod 진입
kubectl exec -it non-web -- bash

# non-web에서 app 서버 호출
$root@non-web:/# curl 10.42.0.172
# curl: (7) Failed to connect to 10.42.0.172 port 80: Connection refused

$root@non-web:/# exit
```

#### DB 접근 Pod

```yaml
# vi K8S/CH05/db-accessable.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-accessable              # 네트워크 정책 이름
  namespace: default               # 적용 대상 네임스페이스
spec:
  podSelector:
    matchLabels:
      run: db                      # 이 정책이 적용될 대상 Pod 선택 (run=db)
  ingress:
  - from:
    - podSelector:
        matchLabels:
          db-accessable: "true"    # 이 라벨을 가진 Pod만 허용
    ports:
    - protocol: TCP
      port: 80                     # 허용 포트: TCP 80
```

```bash
kubectl apply -f K8S/CH05/db-accessable.yaml
# networkpolicy.networking.k8s.io/db-accessable created

# run=db 이라는 라벨을 가진 DB 생성
kubectl run db --image nginx
# pod/db created

# Pod IP 확인
kubectl get pod -owide
# NAME      READY  STATUS    RESTARTS   AGE   IP            NODE     ..
# web       1/1    Running   0          34s   10.42.0.169   master   ..
# non-web   1/1    Running   0          32s   10.42.0.170   master   ..
# client    1/1    Running   0          28s   10.42.0.171   worker   ..
# app       1/1    Running   0          28s   10.42.0.172   worker   ..
# db        1/1    Running   0          28s   10.42.0.173   worker   ..

# app Pod 진입
kubectl exec -it app -- bash

# db로 연결 확인
$root@app:/# curl 10.42.0.173
# curl: (7) Failed to connect to 10.42.0.173 port 80: Connection refused

# app Pod 종료
$root@app:/# exit

# db-accessable=true 라벨 추가
# Labels:           run=app -> 에서  db-accessable=true 추가
kubectl describe pod app
kubectl label pod app db-accessable=true

# Labels에 추가 확인
kubectl describe pod app
# pod/app labeled
```
##### ✅ 명령어 설명
| 구성 요소                | 설명                                    |
| -------------------- | ------------------------------------- |
| `kubectl`            | Kubernetes CLI 명령어 도구                 |
| `label`              | 리소스에 라벨을 추가/수정하는 명령                   |
| `pod`                | 대상 리소스 유형 (Pod)                       |
| `app`                | 대상 Pod 이름 (즉, 이름이 `app`인 Pod에 라벨을 설정) |
| `db-accessable=true` | 설정할 라벨 키와 값 (`db-accessable: "true"`) |

```bash
# app Pod 진입
kubectl exec -it app -- bash

# db로 연결 확인
$root@app:/# curl 10.42.0.173
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...

$root@app:/# exit
```

#### DMZ zone 연결
##### 🔐 DMZ Zone이란?
* DMZ는 내부 네트워크와 외부 네트워크 사이에 위치한 중간 네트워크 영역입니다.
* 주로 공개 서비스(예: 웹 서버, 프록시 서버, API 게이트웨이 등)를 두며,
* 외부에서 접근은 가능하지만 내부 네트워크로의 직접 접근은 제한됩니다.
* Kubernetes에서는 NetworkPolicy를 이용하여 DMZ를 구현할 수 있습니다.

```yaml
#웹서버의 N/W보안을 높히고자 DMZ를 만들어 프록시 서버를 거쳐서 웹서로 들어오도록 수정
# vi K8S/CH05/allow-dmz.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-dmz                   # 정책 이름
  namespace: default               # 정책이 적용되는 네임스페이스
spec:
  podSelector:                     # 대상 Pod (정책이 적용될 대상)
    matchLabels:
      run: web                     # 라벨이 run=web 인 Pod (ex: nginx 웹서버)
  ingress:
  - from:
    - namespaceSelector:           # 접속을 허용할 소스 네임스페이스
        matchLabels:
          zone: dmz               # zone=dmz 라벨이 붙은 네임스페이스만 허용
    ports:
    - protocol: TCP
      port: 80                    # TCP 80 포트만 허용
```

```bash
kubectl delete networkpolicy web-open
# networkpolicy.networking.k8s.io/web-open deleted

kubectl create ns dmz
# namespace/dmz created

kubectl label namespace dmz zone=dmz
# namespace/dmz labeld
```

```bash
kubectl apply -f K8S/CH05/allow-dmz.yaml
# networkpolicy.networking.k8s.io/allow-dmz created

# DMZ 네임스페이스 Proxy 서버 생성
kubectl run proxy --image nginx -n dmz
# pod/proxy created

# dmz 네임스페이스 Pod 조회
kubectl get pod -owide -n dmz
# NAME      READY   STATUS    RESTARTS   AGE     IP            NODE 
# proxy     1/1     Running   0          34s     10.42.0.183   master

# proxy Pod 진입
kubectl exec -it proxy -n dmz -- bash

# dmz 네임스페이스에 있는 proxy 서버에서는 web 서버로 통신이 잘 됩니다.
$root@proxy:/# curl 10.42.0.169   # 웹서버 IP
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...

# proxy Pod 종료
$root@proxy:/# exit

# default 네임스페이스의 client Pod로 들어가서 다시 web 서버로 호출합니다.
kubectl exec -it client -- bash

# web 서버로 호출
$root@client:/# curl 10.42.0.169
# curl: (7) Failed to connect to 10.42.0.169 port 80: Connection refused

$root@client:/# exit
```

### 13.3.5 네트워크 구성 - Egress
* Egress는 Pod가 외부로 나가는 네트워크 트래픽을 제어하는 기능

#### dev 네임스페이스 외의 아웃바운드 차단

```yaml
# vi K8S/CH05/dont-leave-dev.yaml
# dev 외에 아웃바운드 차단
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: dont-leave-dev       # 정책 이름
  namespace: dev             # 정책이 적용될 네임스페이스
spec:
  podSelector: {}            # dev 네임스페이스 내의 모든 Pod에 적용
  policyTypes:
  - Egress                   # Egress 트래픽만 제어 (외부로 나가는 트래픽)
  egress:
  - to:
    - podSelector: {}        # 같은 네임스페이스(dev)의 모든 Pod에만 Egress 허용
```

```bash
# dev 네임스페이스 생성
kubectl create ns dev
# namespace/dev created

# Egress 네트워크 정책 생성
kubectl apply -f K8S/CH05/dont-leave-dev.yaml
# networkpolicy.networking.k8s.io/dont-leave-dev created

kubectl run dev1 --image nginx -n dev
# pod/dev1 created

kubectl run dev2 --image nginx -n dev
# pod/dev2 created

# Pod IP 확인
kubectl get pod -owide -n dev
# NAME      READY   STATUS    RESTARTS   AGE     IP            NODE 
# dev1      1/1     Running   0          34s     10.42.0.191   master 
# dev2      1/1     Running   0          32s     10.42.0.192   master 

# dev1 Pod 진입
kubectl exec -it dev1 -n dev -- bash

# dev1에서 dev2로 호출
$root@dev1:/# curl 10.42.0.192
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...

# dev1에서 proxy 서버로 호출 (proxy 서버IP: 10.42.0.183)
$root@dev1:/# curl 10.42.0.183
# curl: (7) Failed to connect to 10.42.0.183 port 80: Connection refused

$root@dev1:/# exit
```

#### Metadata API 접근 금지
* 클라우드 서비스에서 제공하는 서버의 경우 특정IP를 이용하여 인스턴스의 메타데이터에 접근할 수 있다
* 보안상 사용자들이 직접 metadata에 접근하지 못하도록 IP를 차단하고 싶을 뗴 ipBlock을 가지고 특정 IP대역을 차단할 수 있다.

###### AWS EC2가 있을 떄 실습결과 확인 가능, 예를 들어 EC2 ip가  169.254.169.254로 가정
```bash
# instance id 확인
curl 169.254.169.254/1.0/meta-data/instance-id
# i-0381e52ee2cxxx
```

```yaml
# vi K8S/CH05/block-metadata.yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: block-metadata                # 네트워크 정책 이름
  namespace: default                  # 적용할 네임스페이스 (여기서는 default)
spec:
  podSelector: {}                     # default 네임스페이스의 모든 Pod에 적용
  policyTypes:
  - Egress                            # Egress 정책을 정의 (명시해야 효과적 적용됨)
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0               # 외부 전체 주소 범위
        except:
        - 169.254.169.254/32          # 메타데이터 API 주소는 예외 처리 → 즉, 차단됨
```
##### ✅ 정리
1. cidr: 0.0.0.0/0은 모든 외부 IP로 나가는 트래픽을 의미합니다.
1. except: 169.254.169.254/32는 메타데이터 API 주소를 제외하므로, 이 IP로의 접근은 차단됩니다.
1. policyTypes: [Egress]를 반드시 명시해야 egress 정책이 활성화됩니다. (생략하면 Ingress만 적용됨)

```bash
# client Pod 진입
kubectl exec -it client -- bash

# IP를 이용하여 metadata를 확인할 수 있습니다.
$root@client:/# curl 169.254.169.254/1.0/meta-data/instance-id
# i-0381e52ee2cxxx

# client Pod 종료
$root@client:/# exit

# IP 대역 차단 네트워크 정책을 생성합니다.
kubectl apply -f K8S/CH05/block-metadata.yaml
# networkpolicy.networking.k8s.io/block-metadata created

# client Pod 진입
kubectl exec -it client -- bash

# 동일한 명령에 대해서 연결이 막혔습니다.
$root@client:/# curl 169.254.169.254/1.0/meta-data/instance-id
# curl: (7) Failed to connect to 169.254.169.254 port 80: Connection refused

exit
```

### 13.3.6 `AND` & `OR` 조건 비교
* NetworkPolicy에서 AND 및 OR 조건을 사용하는 방식은 조금 특별합니다. 
* 일반적인 프로그래밍 언어처럼 논리 연산자를 직접 사용하지 않고, YAML 구조를 통해 표현

#### `AND` 조건
* AND는 하나의 matchLabels 또는 matchExpressions 블록 안에서 여러 조건을 정의할 때 사용됩니다. 
* 즉, 모든 조건이 동시에 일치해야 합니다.

```yaml
# from 속서에 리스트원소를 2개의 podSelector로 선언할 때 `circle AND red`
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: and-condition
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          shape: circle
    - podSelector:
        matchLabels:
          color: red
```

#### `OR` 조건
* OR는 YAML 배열을 통해 구현됩니다. 즉, 배열의 각 요소 중 하나라도 조건이 맞으면 허용

```yaml
# ingress.from.podSelector 설정은 `circle OR red`
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: or-condition
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          shape: circle
  - from:
    - podSelector:
        matchLabels:
          color: red
```

### 13.3.7 네트워크 정책 전체 스펙
* 지금까지의 NetworkPolicy의 전체 내용

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: full-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: dev
    - podSelector:
        matchLabels:
          role: web
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 53
```

### Clean up

```bash
kubectl delete pod --all
kubectl delete networkpolicy --all -A
kubectl delete ns dmz
kubectl delete ns dev
```