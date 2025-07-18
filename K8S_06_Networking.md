# 6. 쿠버네티스 네트워킹
>* 쿠버네티스 네트워킹은 Service리소스가 담당
>* Service리소스도 Yaml형식으로 정의
>* Service리소스는 Pop IP와는 다른 독자적인 IP를 부여 받아 서비스의 EndPoint를 제공
>* 라벨링시스템을 통해 Pod로 트레픽을 전달
>* Service리소스는 Pop앞단에 위치하여 마치 로드발렌서처럼 동작

## 쿠버네티스 네트워킹 기본 개념
1. Pod 간 통신 (Pod-to-Pod communication)
* 클러스터 내 모든 Pod는 서로 직접 통신 가능해야 한다.
* 기본적으로 모든 Pod는 고유한 IP 주소를 가진다.
* Pod IP는 클러스터 내에서 유일하며, 서비스 검색 없이도 IP로 통신 가능.
* 네트워크 정책(NetworkPolicy)으로 통신 허용/차단 가능.

2. 서비스(Service) 네트워킹
* Pod IP는 변할 수 있으므로, 서비스가 추상화 역할을 한다.
* 서비스는 고정된 IP와 DNS 이름을 제공해, Pod 집합에 대한 접근을 가능케 함.
* ClusterIP (클러스터 내부), NodePort, LoadBalancer 등 서비스 타입별 접근 경로 제공.

3. 노드 간 네트워킹 (Node-to-Node communication)
* 노드들 간에는 네트워크가 연결되어야 하고, Pod IP가 어디서든 라우팅 가능해야 한다.
* 네트워크 플러그인(CNI 플러그인)이 이 역할 담당.

## 쿠버네티스 네트워킹 기본 원칙 (3대 원칙)
* 모든 Pod는 서로 직접 통신 가능해야 한다.
* 모든 노드는 모든 Pod IP와 통신 가능해야 한다.
* 호스트 네트워크는 별도 관리되고, Pod 네트워크와 격리되어야 한다.

## 주요 네트워크 구성 요소
| 구성 요소                                 | 설명                                       |
| ------------------------------------- | ---------------------------------------- |
| **Pod IP**                            | 각 Pod가 부여받는 유니크 IP                       |
| **Service IP (ClusterIP)**            | 클러스터 내에서 고정된 서비스 접근용 IP                  |
| **CNI (Container Network Interface)** | 네트워크 플러그인 (Calico, Flannel, Weave, etc.) |
| **NetworkPolicy**                     | 네트워크 트래픽의 허용/차단 정책                       |
| **Ingress**                           | 클러스터 외부에서 내부 서비스로의 HTTP/HTTPS 접근         |


## 네트워크 플러그인 (CNI) 예시
* Calico: 네트워크 정책 및 보안에 강점
* Flannel: 간단하고 빠른 Overlay 네트워크 제공
* Weave Net: 손쉬운 설치와 자동 메쉬 네트워킹
* Cilium: eBPF 기반 고성능 보안과 네트워킹

## Pod와 서비스 간 통신 흐름
* Pod → Pod 직접 IP 접근 가능
* Pod → Service (ClusterIP) → Service의 Endpoint(실제 Pod들)로 트래픽 전달
* 외부 → NodePort 또는 LoadBalancer → 내부 Service → Pod

### 요약
* 쿠버네티스 네트워킹은 Pod IP 기반 통신 보장
* 서비스는 안정적인 접속을 위한 추상화
* CNI 플러그인이 실제 네트워크 연결을 담당
* 네트워크 정책으로 보안 및 접근 제어 가능

## 6.1 Service 소개
* 쿠버네티스의 **Service(서비스)**는 Pod 집합에 대한 네트워크 접근을 안정적이고 일관되게 제공하는 리소스.
* 서비스는 Pod가 죽거나 재시작되어도 변하지 않는 IP와 DNS 이름을 제공하여, 클라이언트가 항상 동일한 방법으로 접근할 수 있게 한다.

### 🔧 왜 Service가 필요한가?
* Pod는 재시작 시 IP가 바뀜 → 직접 접근 불가능
* Service는 Pod 앞에 고정된 접점(IP, DNS)을 제공
* 여러 Pod(레플리카)에 대해 로드밸런싱 역할 수행

### ✅ Service의 종류
| 종류                  | 설명                                                      |
| ------------------- | ------------------------------------------------------- |
| **ClusterIP (기본값)** | 클러스터 내부에서만 접근 가능 (내부 서비스용)                              |
| **NodePort**        | 외부에서 노드의 포트를 통해 접근 가능                                   |
| **LoadBalancer**    | 클라우드 환경에서 외부 로드밸런서를 자동 생성                               |
| **ExternalName**    | 서비스 이름을 외부 도메인으로 매핑 (DNS alias)                         |
| **Headless**        | 클러스터 DNS를 통해 Pod IP 목록 직접 반환 (LoadBalancer 없이 직접 접근 가능) |

### 📄 ClusterIP 예제 (기본 타입)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app              # 이 라벨을 가진 Pod들을 대상으로
  ports:
  - protocol: TCP
    port: 80                 # 클라이언트가 접근할 서비스 포트
    targetPort: 8080         # Pod 내부에서 실제로 사용하는 포트
# my-service라는 이름으로 클러스터 내 DNS에 등록됨
# (예: http://my-service.default.svc.cluster.local)
```

### 🌐 NodePort 예제
```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080   # (30000~32767 범위)
# 클러스터 외부에서: http://<노드IP>:30080 으로 접근 가능
```

### LoadBalancer 예제 (클라우드 환경에서만)
```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```
* 클라우드에서 외부 IP가 자동으로 부여됨
* 외부 사용자 → LoadBalancer IP → 서비스 → Pod

### 🔍 서비스 검색 (Service Discovery)
* 모든 Service는 클러스터 내 DNS 이름을 갖습니다.
  - 예: my-service.default.svc.cluster.local
* 애플리케이션은 이 이름을 통해 서비스에 접근

### 🎯 요약
| 기능     | 설명                                 |
| ------ | ---------------------------------- |
| 로드밸런싱  | 동일 레이블을 가진 여러 Pod에 트래픽 분산          |
| IP 안정성 | Pod가 죽거나 재시작돼도 서비스 IP는 유지          |
| DNS 제공 | 내부 DNS 이름으로 서비스 접근 가능              |
| 외부 노출  | NodePort, LoadBalancer로 외부에서 접근 가능 |

### 🔍 Pod vs Service 비교표
* Pod는 언제든지 종료/삭제될 수 있는 리소스인 특징으로 불안정한 자원이다.
* 이러한 문제를 해결하기 위해 Pod의 생명주기와는 상관없이 안정적인 서비스를 제공하는 Service리소스를 사용한다.

| 항목        | **Pod**                              | **Service**                               |
| --------- | ------------------------------------ | ----------------------------------------- |
| **정의**    | 하나 이상의 컨테이너를 실행하는 가장 작은 배포 단위        | Pod 집합에 대한 네트워크 접근을 제공하는 추상 리소스           |
| **역할**    | 실제 애플리케이션이 동작하는 컨테이너 환경              | 외부나 내부에서 Pod에 안정적으로 접근하기 위한 엔드포인트 제공      |
| **IP**    | Pod마다 고유한 IP 부여됨 (비영구적, 재시작 시 변경됨)   | 고정된 ClusterIP 할당됨 (서비스 생명주기 동안 유지)        |
| **접근 방법** | 직접 접근 가능하지만 IP가 변경될 수 있어 불안정함        | DNS 또는 IP를 통해 안정적인 접근 가능                  |
| **내부 통신** | Pod 간 직접 통신 가능 (같은 네임스페이스 또는 클러스터 내) | 다른 Pod나 외부 트래픽이 안정적으로 Pod에 도달하도록 중간 역할 수행 |
| **로드밸런싱** | 개별 Pod로 직접 접근 시 로드밸런싱 불가능            | 동일 라벨을 가진 여러 Pod에 대해 자동 로드밸런싱 지원          |
| **수명**    | 일시적일 수 있고, 재시작 시 삭제/재생성됨             | 상대적으로 안정적이고 Pod 변경과 관계없이 계속 유지됨           |
| **구성 대상** | 컨테이너 (애플리케이션 자체)                     | 특정 레이블을 가진 Pod (애플리케이션을 감싸는 접점)           |

### 🎯 예시로 이해하기
#### ✅ Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: nginx
# → 실제로 컨테이너(Nginx)가 동작하는 실행 단위
```

#### ✅ Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
# → 레이블이 app: my-app인 Pod들을 찾아 하나의 IP로 묶어 접근 가능하게 함
# (클러스터 외부 또는 내부에서)
```
### 🔑 정리
* Pod는 "무엇을 실행할지" 정의한 것 (실행 단위)
* Service는 "어떻게 접근할지" 정의한 것 (네트워크 접근 단위)
* 둘은 함께 사용되어, 쿠버네티스 애플리케이션의 확장성, 안정성, 가용성을 보장합니다.

## 🎯 실습

```bash
kubectl run mynginx --image nginx
# pod/mynginx created

# Pod IP는 사용자마다 다릅니다.
kubectl get pod -owide
# NAME     READY   STATUS     RESTARTS   AGE   IP           NODE    ...
# mynginx  1/1     Running    0          12d   10.42.0.26   master  ...

kubectl exec mynginx -- curl -s 10.42.0.26
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
#     body {
# ...
```

### 6.1.3 Service 첫 만남
```bash
vi K8S/CH05/myservice.yaml
```
```yaml
# myservice.yaml
# mynginx라는 Pod에 대한 접근을 가능하게 해주는 ClusterIP 서비스를 정의
apiVersion: v1             # 사용하는 Kubernetes API 버전
kind: Service              # 리소스의 종류: Service

metadata:
  labels:
    hello: world           # 서비스에 부여된 라벨 (선택적)
  name: myservice          # 서비스 이름 (DNS 이름으로도 사용됨)

spec:
  ports:                   # 서비스가 노출하는 포트 목록
  - port: 8080             # 클라이언트가 접근할 서비스 포트 (myservice:8080)
    protocol: TCP          # 사용할 네트워크 프로토콜 (기본값은 TCP)
    targetPort: 80         # 서비스가 라우팅할 대상 Pod의 포트 (컨테이너 내부 포트)

  selector:
    run: mynginx           # 이 라벨을 가진 Pod들을 서비스 대상으로 연결
```

#### 라벨 셀렉터를 이용하여 `Pod`를 선택하는 이유

```bash
# 앞에서 살펴 본 Service 리소스를 생성합니다.
kubectl apply -f K8S/CH05/myservice.yaml
# service/myservice created

# 생성된 Service 리소스를 조회합니다.
# Service IP (CLUSTER-IP)를 확인합니다. (예제에서는 10.43.152.73)
kubectl get service  # 축약 시, svc
# NAME          TYPE        CLUSTER-IP     EXTERNAL-IP  PORT(S)    AGE
# kubernetes    ClusterIP   10.43.0.1      <none>       443/TCP    24d
# myservice     ClusterIP   10.43.152.73   <none>       8080/TCP   6s

# Pod IP를 확인합니다. (예제에서는 10.42.0.226)
kubectl get pod -owide
# NAME     READY  STATUS    RESTARTS   AGE   IP            NODE   ...
# mynginx  1/1    Running   0          6s    10.42.0.226   master ...
```

```bash
# curl 요청할 client Pod 생성
kubectl run client --image nginx
# pod/client created

# Pod IP로 접근
kubectl exec client -- curl -s 10.42.0.226

# Service IP로 접근 (CLUSTER-IP)
kubectl exec client -- curl -s 10.43.152.73:8080

# Service 이름 (DNS 주소)로 접근
kubectl exec client -- curl -s myservice:8080
```

```bash
# DNS lookup을 수행하기 위해 nslookup 명령을 설치합니다.
kubectl exec client -- sh -c "apt update && apt install -y dnsutils"
# Hit:1 http://deb.debian.org/debian buster InRelease
# Hit:2 http://security.debian.org/debian-security buster/updates 
# Reading package lists...
# ...

# myservice의 DNS를 조회합니다.
# nslookup myservice	DNS를 통해 myservice의 IP 주소를 조회
kubectl exec client -- nslookup myservice
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73
```

### 6.1.4 `Service` 도메인 주소 법칙

#### ✅ 기본 도메인 주소 형식
```bash
<service-name>.<namespace>.svc.cluster.local
```
| 구성 요소           | 설명                         |
| --------------- | -------------------------- |
| `service-name`  | Service 이름 (metadata.name) |
| `namespace`     | Service가 속한 네임스페이스         |
| `svc`           | 서비스 타입 도메인 (`Service`)     |
| `cluster.local` | 클러스터 기본 도메인 (디폴트 DNS 서픽스)  |

### 🌐 요약
| 위치에서 접근          | 도메인 이름 예시                                    |
| ---------------- | -------------------------------------------- |
| 같은 네임스페이스        | `myservice`                                  |
| 다른 네임스페이스        | `myservice.default`, `myservice.default.svc` |
| 전체 FQDN (완전한 주소) | `myservice.default.svc.cluster.local`        |


```bash
# Service의 전체 도메인 주소를 조회합니다.
kubectl exec client -- nslookup myservice.default.svc.cluster.local
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73

# Service의 도메인 주소 일부를 생략하여 조회합니다.
kubectl exec client -- nslookup myservice.default
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73

# Service 이름만 사용해도 참조가 가능합니다.
kubectl exec client -- nslookup myservice
# Server:         10.43.0.10
# Address:        10.43.0.10#53
# 
# Name:   myservice.default.svc.cluster.local
# Address: 10.43.152.73
```

### 6.1.5 클러스터 DNS 서버
<img src="./images/kubernets install/kubernetes-cluster-architecture.svg" width="600" height="400">

#### 🔍 클러스터 DNS 서버란?
* Kubernetes 클러스터에서 DNS 서버는 클러스터 내 모든 Pod가 이름 기반으로 서로 통신할 수 있도록 해주는 내부 DNS 시스템. 
* 이 기능은 CoreDNS 또는 kube-dns가 기본적으로 제공하며, Service, Pod, 네임스페이스 등의 이름을 자동으로 DNS로 변환
* 클러스터 내부에서 DNS 질의(예: myservice.default.svc.cluster.local)를 처리하는 내부 네임서버.
* 대부분의 클러스터에서는 CoreDNS가 기본 설치되어 있다.
* kubelet은 각 Pod가 생성될 때 해당 DNS 서버 주소를 /etc/resolv.conf에 자동 설정.
* Service이름을 도메인주소로 사용이 가능한 이유는 바로 쿠버네티스에서 제공하는 DNS서버가 있기 때문이다.


#### ✅ 기본 구조
```css
[Pod] --> [resolv.conf → CoreDNS] --> [DNS 질의 처리]
```

#### 🧪 Pod 내 DNS 확인 방법
```bash
kubectl exec -it <your-pod> -- cat /etc/resolv.conf
```

#### 🔧 클러스터 DNS가 필요한 이유
| 기능                     | 설명                                                          |
| ---------------------- | ----------------------------------------------------------- |
| Service 이름으로 접근 가능     | IP 없이 `myservice`로 접근                                       |
| 네임스페이스 기반 주소 지원        | `myservice.default` 등 사용 가능                                 |
| StatefulSet 등에서 DNS 생성 | Pod마다 고유한 도메인 생성 가능 (`web-0.web.default.svc.cluster.local`) |

#### 🔧 kubelet이란?
* kubelet은 Kubernetes 노드에서 실행되는 핵심 에이전트입니다. 
* 쿠버네티스 클러스터에서 각 워커 노드에 설치되며, Pod의 생성, 상태 확인, 실행, 삭제 등의 컨테이너 수명 주기 관리를 담당합니다.
| 항목       | 설명                               |
| -------- | -------------------------------- |
| 역할       | 워커 노드의 **Pod 관리자**               |
| 위치       | 모든 **노드에서 실행**                   |
| 주요 기능    | Pod 스펙 확인, 컨테이너 실행 및 상태 모니터링     |
| 통신 대상    | Kubernetes API Server            |
| 관련 구성 파일 | `/var/lib/kubelet/config.yaml` 등 |

##### 📌 kubelet의 주요 역할
| 기능                 | 설명                                                                           |
| ------------------ | ---------------------------------------------------------------------------- |
| Pod 실행 관리          | API Server로부터 할당된 Pod 사양을 받고, 컨테이너 런타임(Docker, containerd 등)을 통해 실행          |
| 상태 보고              | 각 Pod 및 컨테이너 상태를 주기적으로 API 서버에 보고 (`Ready`, `Running`, `CrashLoopBackOff` 등) |
| 상태 확인              | Liveness/Readiness Probe 실행 및 처리                                             |
| 로그 및 이벤트 수집        | 컨테이너 상태 이상 시 로그/이벤트를 기록                                                      |
| CSI, Volume 마운트 관리 | PV, PVC 등 볼륨 연결 및 마운트 처리                                                     |

##### 🧱 kubelet 구조 간단도
```scss
[Kubernetes API Server]
          ↓
       [kubelet] ←→ [container runtime] (e.g. containerd)
          ↓
       [Pod/Container]
```

##### 🔍 kubelet 관련 명령 예시
```bash
# 현재 노드에서 실행 중인 kubelet 상태 확인 (Linux 기준)
systemctl status kubelet

# kubelet 설정 확인 (보통 kubeadm 클러스터의 경우)
cat /var/lib/kubelet/config.yaml
```
##### 💡 참고 설정 항목 (config.yaml 예)
```yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
```
##### 📎 kubelet이 없으면?
* Pod가 실행되지 않음
* API Server의 명령이 전달되지 않음
* 클러스터 상태가 NotReady로 표시됨


```bash
# 로컬 호스트 서버의 DNS 설정이 아닌 Pod의 DNS 설정을 확인합니다.
kubectl exec client -- cat /etc/resolv.conf
# nameserver 10.43.0.10
# search default.svc.cluster.local svc.cluster.local cluster.local ap-northeast-2.compute.internal
# options ndots:5
```

```bash
# kube-system 네임스페이스 내 모든 Service 목록을 조회
kubectl get svc -n kube-system
# NAME      TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S) 
# kube-dns  ClusterIP  10.43.0.10   <none>       53/UDP,53/TCP,9153/TCP 

# ✅ kube-system 네임스페이스란?
# * 쿠버네티스 클러스터의 핵심 컴포넌트(Pod, Service 등) 가 실행되는 시스템 전용 네임스페이스입니다.
#   - 예: kube-dns, metrics-server, kube-proxy 등이 포함됩니다.
```
##### 🔍 예시 출력 필드 설명
| 항목           | 설명                                       |
| ------------ | ---------------------------------------- |
| `NAME`       | 서비스 이름 (예: `kube-dns`, `metrics-server`) |
| `TYPE`       | 서비스 유형 (`ClusterIP`, `NodePort` 등)       |
| `CLUSTER-IP` | 클러스터 내 IP 주소 (내부 DNS 접근 시 사용)            |
| `PORT(S)`    | 서비스가 사용하는 포트                             |
| `AGE`        | 생성된 지 얼마나 되었는지                           |
##### 🔧 주요 서비스 설명
| 서비스 이름                  | 역할                             |
| ----------------------- | ------------------------------ |
| `kube-dns` 또는 `coredns` | 내부 DNS 해석 기능 제공                |
| `metrics-server`        | Pod/노드 리소스 사용량 수집 (HPA 등에서 사용) |
| `kube-proxy` (간혹 존재)    | 클러스터 네트워크 프록시 역할               |


```bash
# kube-dns 서비스의 라벨 정보까지 포함하여 상세 조회하는 명령
kubectl get svc kube-dns -nkube-system --show-labels
# NAME       TYPE         CLUSTER-IP  ...    AGE   LABELS
# kube-dns   ClusterIP    10.43.0.10  ...    46h   k8s-app=kube-dns,...
```
##### 🔍 명령어 구성 설명
| 항목                | 의미                      |
| ----------------- | ----------------------- |
| `kubectl get svc` | Service 리소스를 조회         |
| `kube-dns`        | 대상 서비스 이름               |
| `-n kube-system`  | `kube-system` 네임스페이스 지정 |
| `--show-labels`   | 라벨 정보를 함께 출력            |

###### 주요 라벨 예시:
| 라벨 키                                 | 설명                              |
| ------------------------------------ | ------------------------------- |
| `k8s-app=kube-dns`                   | 애플리케이션 이름 (CoreDNS 또는 kube-dns) |
| `kubernetes.io/name=CoreDNS`         | DNS 시스템의 공식 이름                  |
| `kubernetes.io/cluster-service=true` | (이전 버전에서 사용) 클러스터 핵심 서비스 식별용    |


```bash
# kubernetes에서 kube-dns 관련 Pod들만 필터링하여 조회
kubectl get pod -n kube-system -l k8s-app=kube-dns
# NAME              READY   STATUS    RESTARTS   AGE
# coredns-6c6bb68   1/1     Running   0          46h
```
###### 🔍 구성 설명
| 옵션                    | 설명                                     |
| --------------------- | -------------------------------------- |
| `kubectl get pod`     | Pod 리소스 조회                             |
| `-n kube-system`      | `kube-system` 네임스페이스 대상                |
| `-l k8s-app=kube-dns` | 라벨 셀렉터로 `k8s-app=kube-dns`가 붙은 Pod만 조회 |


## 6.2 Service 종류
### ✅ Kubernetes Service의 종류
| 유형                                       | 설명                                                                  |
| ---------------------------------------- | ------------------------------------------------------------------- |
| **ClusterIP** *(기본값)*                    | 클러스터 내부 IP를 통해 **내부 통신만 가능**<br>예: `frontend → backend`             |
| **NodePort**                             | 모든 노드의 포트를 열어 외부에서 접근 가능<br>예: `노드IP:노드포트 → 서비스`                    |
| **LoadBalancer**                         | 클라우드에서 **외부 로드밸런서 IP**를 통해 접근<br>예: AWS, GCP, Azure                 |
| **ExternalName**                         | 외부 DNS 이름으로 프록시<br>실제 외부 호스트를 가리킴                                   |
| **Headless Service** (`ClusterIP: None`) | 클러스터 DNS에서 **Pod 개별 IP** 반환 (로드밸런싱 X)<br>예: StatefulSet, DNS 라운드로빈용 |

### 📌 예시 비교
| 유형           | 외부 접근  | ClusterIP 생성 | 용도                  |
| ------------ | ------ | ------------ | ------------------- |
| ClusterIP    | ❌ 내부만  | ✅ 생성됨        | 내부 통신용              |
| NodePort     | ✅ 가능   | ✅ 생성됨        | 개발·디버깅용, 소규모 외부 노출  |
| LoadBalancer | ✅ 클라우드 | ✅ 생성됨        | 클라우드 프로덕션 환경        |
| ExternalName | ✅ 가능   | ❌ 생성 안 됨     | 외부 API, DB 등 사용     |
| Headless     | ❌      | ❌ (`None`)   | 직접 Pod IP 사용 (DB 등) |

### 🚀 Tip
* 클러스터 외부 노출이 필요 없다면 ClusterIP로 충분
* 클라우드 환경에서는 LoadBalancer로 외부 접근을 간편하게 구성
* StatefulSet이나 직접 DNS 조회가 필요하면 Headless

### 6.2.1 ClusterIP
* Service리소스의 가장 기본이 되는 타입
* 별다른 타입 지정을 하지 않으면 기본적으로 ClustIP로 설정 된다.
* 클러스터 내부에서만 접근 가능한 가상 IP 주소를 제공하는 서비스
* 외부에서는 접근할 수 없으며, Pod 간 통신, 즉 내부 마이크로서비스 간 통신에 주로 사용

#### ✅ ClusterIP 특징 요약
| 항목           | 설명                                                                   |
| ------------ | -------------------------------------------------------------------- |
| **접근 가능 범위** | 오직 클러스터 내부에서만 접근 가능 (`kubectl exec`, 내부 Pod 등)                       |
| **기본 타입**    | Service를 생성할 때 별도 지정하지 않으면 `ClusterIP`로 생성됨                          |
| **가상 IP**    | 클러스터 내부 DNS에 의해 자동으로 IP 할당됨 (`my-service.default.svc.cluster.local`) |
| **로드밸런싱**    | 동일 라벨을 가진 여러 Pod에 트래픽 분산                                             |

#### 📄 ClusterIP 서비스 생성

```bash
# ClusterIP 타입의 Service와 함께 Pod을 정의하는 YAML을 생성하는 예제
kubectl run cluster-ip --image nginx --expose --port 80 \
    --dry-run=client -o yaml > K8S/CH05/cluster-ip.yaml

vi K8S/CH05/cluster-ip.yaml
```

##### 🧾 명령어 설명
| 옵션                       | 설명                                            |
| ------------------------ | --------------------------------------------- |
| `kubectl run cluster-ip` | 이름이 `cluster-ip`인 Pod을 생성하려는 명령               |
| `--image nginx`          | 사용할 컨테이너 이미지 (nginx)                          |
| `--expose`               | Pod을 노출하는 **Service 리소스**도 생성 (기본: ClusterIP) |
| `--port 80`              | Service가 노출할 포트                               |
| `--dry-run=client`       | 실제 생성하지 않고 **출력만** 수행                         |
| `-o yaml`                | YAML 형식으로 출력                                  |
| `> cluster-ip.yaml`      | 결과를 `cluster-ip.yaml` 파일로 저장                  |

##### 📦 cluster-ip.yaml 구성
* 이 파일에는 두 개의 리소스가 정의되어 있습니다:
  1. Service (ClusterIP 타입)
  2. Pod (nginx 컨테이너)

```yaml
# cluster-ip.yaml
apiVersion: v1
kind: Service
metadata:
  name: cluster-ip
spec:
  # type: ClusterIP  # 기본값이므로 생략 가능
  ports:
  - port: 8080             # 클러스터 내부에서 노출할 포트
    protocol: TCP
    targetPort: 80         # 실제 Pod에서 열려 있는 컨테이너 포트
  selector:
    run: cluster-ip        # 해당 라벨을 가진 Pod로 트래픽 전달

# ✅ 설명:
# ClusterIP 타입은 Kubernetes 기본 서비스 타입이며, 클러스터 내부에서만 접근 가능
# port: 8080 → 클러스터에서 서비스되는 포트
# targetPort: 80 → 컨테이너 내부에서 노출되는 포트
# selector.run: cluster-ip → 아래의 Pod에 연결
---
apiVersion: v1              # API 버전 (v1은 기본 Core API 그룹)
kind: Pod                   # 생성하려는 리소스의 종류 (여기서는 Pod)
metadata:                   # 메타데이터 정보 블록
  labels:                   # 라벨 지정 (Pod을 식별하는 데 사용)
    run: cluster-ip         # 'run'이라는 키에 'cluster-ip' 값을 가지는 라벨
  name: cluster-ip          # Pod의 이름
spec:                       # Pod의 스펙 정의
  containers:               # Pod 안에 포함된 컨테이너 목록
  - image: nginx            # 사용할 컨테이너 이미지 (nginx 웹 서버)
    name: nginx             # 컨테이너 이름
    ports:                  # 컨테이너에서 사용하는 포트 목록
    - containerPort: 80     # 컨테이너가 리스닝할 포트 번호 (HTTP 기본 포트)


# ✅ 설명:
# 이름이 cluster-ip인 Pod이며, nginx 이미지 사용
# 컨테이너에서 80번 포트를 열고 있음
# labels.run: cluster-ip 이므로 위 Service와 연결됨
```

```bash
kubectl apply -f K8S/CH05/cluster-ip.yaml
# service/cluster-ip created
# pod/cluster-ip created

kubectl get svc cluster-ip -oyaml | grep type
#  type: ClusterIP

kubectl describe svc cluster-ip
kubectl exec client -- curl -s cluster-ip
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```
🧾 명령어 구성 설명
| 부분                    | 의미                                                                           |
| --------------------- | ---------------------------------------------------------------------------- |
| `kubectl exec client` | `client`라는 Pod 안에서 명령어 실행                                                    |
| `--`                  | `kubectl` 옵션과 실제 컨테이너 명령어를 구분                                                |
| `curl -s cluster-ip`  | `client` Pod 안에서 `curl` 명령을 사용해 `cluster-ip`라는 이름의 서비스에 요청 (`-s`는 silent 모드) |


### 6.2.2 NodePort
* NodePort는 Kubernetes의 Service 유형 중 하나로, 클러스터 외부에서 클러스터 내부의 서비스(예: 웹서버)에 접근할 수 있도록 해주는 방법
* NodePort는 특히 개발, 테스트, 간단한 외부 접근용 환경에서 자주 사용된다.

### 📌 NodePort란?
* 기본 개념: NodePort는 **각 노드(Node)**의 특정 포트를 열어 외부에서 해당 포트를 통해 클러스터 내부의 서비스로 요청이 전달되도록 해줍니다.
* 클러스터에 있는 **모든 노드의 IP 주소와 지정된 포트(NodePort)**를 통해 서비스에 접근할 수 있습니다.

### 🛠️ 동작 방식
1. Service 정의 시 type을 NodePort로 설정:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80           # 클러스터 내부에서 접근할 서비스 포트
      targetPort: 80     # 실제 컨테이너가 리스닝하는 포트
      nodePort: 30080    # 외부에서 접근할 수 있는 노드 포트 (30000~32767 사이)
```
2. 위 설정이 적용되면, 클러스터의 **모든 노드 IP + nodePort(예: 30080)**로 요청 시, 해당 요청이 my-app을 실행 중인 Pod로 전달됩니다.

### 🔄 흐름 예시
1. 사용자가 웹 브라우저에서 다음 주소로 접속: http://<Node IP>:30080
2. 해당 요청은 NodePort(30080)를 통해 클러스터 내 Service로 전달.
3. Service는 selector에 따라 해당 Pod로 트래픽을 전달.

### ✅ 장점
* 간단하고 빠르게 외부 노출 가능.
* 별도의 로드 밸런서 없이도 외부 접근 가능.

### ⚠️ 단점 및 주의사항
| 항목       | 설명                                            |
| -------- | --------------------------------------------- |
| 🔐 보안    | 노드의 포트를 외부에 직접 노출하므로 방화벽 설정 필요.               |
| 🌐 접근성   | 외부에서 접근하려면 **노드의 공인 IP**를 알아야 함.              |
| 📦 스케일링  | 로드 밸런싱 기능은 있지만, 고급 트래픽 관리에는 한계.               |
| 🔢 포트 제한 | NodePort는 반드시 **30000\~32767** 사이의 포트만 사용 가능. |

### 📘 NodePort vs ClusterIP vs LoadBalancer 요약
| 유형           | 외부 접근 | 설명                              |
| ------------ | ----- | ------------------------------- |
| ClusterIP    | ❌     | 기본 유형. 클러스터 내부에서만 접근 가능.        |
| NodePort     | ✅     | 노드의 포트를 열어 외부 접근 허용.            |
| LoadBalancer | ✅✅    | 클라우드 환경에서 외부 로드 밸런서를 생성해 접근 가능. |

### ✏️ 추가 팁
* nodePort를 명시하지 않으면 Kubernetes가 자동으로 랜덤한 포트(30000~32767 중) 하나를 할당합니다.
* 클러스터 외부에서 접근할 수 있는 가장 간단한 방법이지만, 프로덕션 환경에서는 보통 Ingress 또는 LoadBalancer를 권장합니다.

```bash
vi K8S/CH05/node-port.yaml
```
```yaml
# node-port.yaml
apiVersion: v1                   # Kubernetes API 버전 (v1)
kind: Service                    # 생성할 리소스의 종류: Service
metadata:                        # 메타데이터 블록
  name: node-port                # 서비스의 이름

spec:                            # 서비스의 상세 설정
  type: NodePort                 # Service 타입: 외부 접근이 가능한 NodePort
  ports:                         # 서비스 포트 설정
  - port: 8080                   # 클러스터 내부에서 사용할 포트
    protocol: TCP                # 통신 프로토콜 (기본값: TCP)
    targetPort: 80              # 실제 컨테이너가 리스닝하는 포트
    nodePort: 30080              # 노드에서 외부 접근 시 사용할 포트 (30000~32767 사이에서 지정)
  selector:                      # 이 서비스가 트래픽을 전달할 대상 Pod의 라벨
    run: node-port               # 'run: node-port' 라벨을 가진 Pod에 트래픽 전달

---
apiVersion: v1                   # Kubernetes API 버전 (v1)
kind: Pod                        # 생성할 리소스의 종류: Pod
metadata:                        # 메타데이터 블록
  labels:                        # Pod에 설정할 라벨
    run: node-port               # 'run=node-port' 라벨 (Service의 selector와 일치해야 함)
  name: node-port                # Pod의 이름

spec:                            # Pod의 상세 설정
  containers:                    # 컨테이너 목록
  - image: nginx                 # 사용할 컨테이너 이미지 (nginx 웹 서버)
    name: nginx                  # 컨테이너 이름
    ports:                       # 컨테이너가 열 포트 목록
    - containerPort: 80         # 컨테이너 내부에서 열 포트 (nginx 기본 포트)
```
#### ✅ 이 매니페스트의 주요 포인트
* 클러스터 외부에서 다  `http://<Node IP>:30080` 접근할 수 있습니다:
* 클러스터에 여러 노드가 있으면 어느 노드의 IP든 상관없이 해당 포트로 접속 가능합니다.
* Pod과 Service는 run: node-port 라벨로 연결되어 있습니다.

```bash
kubectl apply -f K8S/CH05/node-port.yaml
# service/node-port created
# pod/node-port created

kubectl get svc
# NAME         TYPE        CLUSTER-IP     EXTERNAL-IP  PORT(S)         AGE
# kubernetes   ClusterIP   10.43.0.1      <none>       443/TCP         26d
# myservice    ClusterIP   10.43.152.73   <none>       8080/TCP        2d
# cluster-ip   ClusterIP   10.43.9.166    <none>       8080/TCP        42h
# node-port    NodePort    10.43.94.27    <none>       8080:30080/TCP  42h
```

```bash
# 트래픽을 전달 받을 Pod가 마스터 노드에 위치합니다.
kubectl get pod node-port -owide
# NAME         READY   STATUS    RESTARTS   AGE   IP           NODE   
# node-port    1/1     Running   0          14m   10.42.0.28   master

kubectl get node # master node
kubectl describe node master
kubectl get node master -o json
kubectl get node master -o json | jq '.'
kubectl get node master -o json | jq '.status'
kubectl get node master -ojsonpath="{}"
kubectl get node master -ojsonpath="{.status}"
kubectl get node master -ojsonpath="{.status.addresses[0]}"
kubectl get node master -ojsonpath="{.status.addresses[0].address}"
MASTER_IP=$(kubectl get node master -ojsonpath="{.status.addresses[0].address}")
# 클러스터의 마스터 노드 IP 주소를 변수 MASTER_IP에 저장하는 쉘 명령

curl $MASTER_IP:30080
# <!DOCTYPE html>
# <html>
# <head>
# ...

# worker1 node
WORKER_IP=$(kubectl get node worker1 -ojsonpath="{.status.addresses[0].address}")
curl $WORKER_IP:30080
# <!DOCTYPE html>
# <html>
# <head>
# ...
```

```bash
# <공인IP> = 192.168.164.131
curl <공인IP>:30080

# 웹 브라우저에서 <공인IP>:30080으로도 확인할 수 있습니다.
```

### 6.2.3 LoadBalancer
* LoadBalancer 타입 서비스는 클라우드 제공자의 로드 밸런서(예: AWS ELB, GCP Cloud Load Balancer, Azure Load Balancer)를 자동으로 프로비저닝해줍니다.
* 외부에서 접근 가능한 공인 IP 주소와 DNS 이름을 자동으로 할당해주어, 클러스터 외부에서 서비스에 쉽게 접속할 수 있게 해줍니다.

#### ⚙️ 동작 방식
1. 사용자가 type: LoadBalancer인 Service 생성
2. 클라우드에서 외부 로드밸런서 인스턴스 프로비저닝
3. 로드밸런서가 서비스와 연결된 Pod로 트래픽 분산
4. 외부 클라이언트가 로드밸런서 IP/DNS로 접속

#### 📌 주요 특징
| 특징             | 설명                  |
| -------------- | ------------------- |
| 🌐 외부 IP 자동 할당 | 클라우드 제공자가 외부 IP 할당  |
| 🔄 자동 로드밸런싱    | Pod들 간 트래픽 분산       |
| 🚪 간편한 외부 접근   | 별도 설정 없이 외부 접근 가능   |
| ☁️ 클라우드 의존성    | 클라우드 환경에서만 지원       |
| 💰 비용 발생       | 클라우드 로드밸런서 비용 발생 가능 |


#### 📝 간단한 예제
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80          # 서비스 포트
      targetPort: 80    # Pod 컨테이너 포트
```
* 위 서비스를 생성하면 클라우드가 자동으로 로드밸런서를 만들어 외부 IP를 할당해 줍니다.
* kubectl get svc my-loadbalancer-service 명령으로 할당된 외부 IP를 확인할 수 있습니다.

#### 🔍 LoadBalancer vs NodePort 비교
| 구분         | LoadBalancer       | NodePort             |
| ---------- | ------------------ | -------------------- |
| 🌐 외부접근    | 클라우드 로드밸런서 통해 가능   | 노드 IP + 노드 포트로 직접 접근 |
| 🛠️ 설정 편의성 | 매우 간편 (클라우드 자동 처리) | 노드마다 직접 포트 노출 필요     |
| 📈 확장성     | 로드밸런서가 트래픽 분산      | 기본적인 분산 지원           |
| 💸 비용      | 클라우드 로드밸런서 비용 발생   | 비용 없음                |
| ☁️ 환경 제약   | 클라우드 환경 전용         | 모든 환경에서 가능           |


#### 📌 실습
```bash
vi K8S/CH05/load-bal.yaml
```
```yaml
# load-bal.yaml
# type: LoadBalancer 설정 시, 클라우드 환경에서는 외부 IP가 자동 할당되어 외부에서 쉽게 접근할 수 있습니다.
# nodePort를 명시해도 되지만 클라우드 환경에서는 보통 자동 할당을 권장합니다.
# selector가 run: load-bal인 Pod에 트래픽을 분산합니다
apiVersion: v1                   # API 버전 (v1)
kind: Service                    # 리소스 종류: Service
metadata:
  name: load-bal                 # 서비스 이름

spec:
  type: LoadBalancer             # 서비스 타입: 클라우드 로드밸런서 프로비저닝
  ports:
  - port: 8080                  # 서비스가 노출하는 포트 (외부 접속 포트)
    protocol: TCP               # 프로토콜 (기본: TCP)
    targetPort: 80              # 실제 Pod 컨테이너가 사용하는 포트
    nodePort: 30088             # 노드 포트 (NodePort와 연동, 지정 가능, 30000~32767 범위)
  selector:
    run: load-bal               # 이 라벨을 가진 Pod에 트래픽을 전달

---
apiVersion: v1                   # API 버전 (v1)
kind: Pod                        # 리소스 종류: Pod
metadata:
  labels:
    run: load-bal               # Pod 라벨 (Service selector와 매칭)
  name: load-bal                 # Pod 이름

spec:
  containers:
  - image: nginx                 # nginx 컨테이너 이미지 사용
    name: nginx                 # 컨테이너 이름
    ports:
    - containerPort: 80         # 컨테이너 내부 포트 (nginx 기본 포트)

```

```bash
kubectl apply -f K8S/CH05/load-bal.yaml
# service/load-bal created
# pod/load-bal created

kubectl get svc load-bal
# NAME       TYPE           CLUSTER-IP   EXTERNAL-IP                       PORT(S)          AGE
# load-bal   LoadBalancer   10.43.21.0   192.168.164.130,192.168.164.131   8080:30088/TCP   3m18s
```
###### ✅ 왜 CLUSTER-IP와 EXTERNAL-IP 범위가 다른가?
| IP 종류                           | 역할 설명                                                                                |
| ------------------------------- | ------------------------------------------------------------------------------------ |
| **CLUSTER-IP** (`10.x.x.x`)     | Kubernetes 클러스터 내부에서만 사용하는 가상 IP입니다. Pod나 다른 Service가 이 주소로 통신할 수 있습니다. 외부에서는 접근 불가. |
| **EXTERNAL-IP** (`192.168.x.x`) | 클러스터 외부(사용자 또는 클라이언트)가 접근할 수 있는 IP입니다. 실제 노드의 IP 주소이거나, 로드밸런서 또는 MetalLB가 할당한 IP입니다. |
* 👉 즉, 두 IP는 용도와 네트워크 계층이 다릅니다.


```bash
 kubectl describe svc load-bal
```
###### 📝 서비스 메타 정보
| 항목              | 설명                                  |
| --------------- | ----------------------------------- |
| **Name**        | 서비스 이름 (`load-bal`)                 |
| **Namespace**   | 서비스가 속한 네임스페이스 (`default`)          |
| **Labels**      | 이 서비스에 설정된 라벨 (`없음`)                |
| **Annotations** | 주석 (메타데이터용 정보, 현재 없음)               |
| **Selector**    | `run=load-bal` 라벨을 가진 Pod에 트래픽을 전달함 |
###### ⚙️ 네트워크 설정
| 항목                                                                                            | 설명                                        |
| --------------------------------------------------------------------------------------------- | ----------------------------------------- |
| **Type**                                                                                      | 서비스 타입: `LoadBalancer` → 외부에서 접근 가능한 서비스  |
| **IP Family Policy**                                                                          | IPv4/IPv6 사용 정책 (`SingleStack`: IPv4만 사용) |
| **IP Families**                                                                               | 사용 중인 IP 패밀리 (`IPv4`)                     |
| **IP** / **IPs**                                                                              | 클러스터 내부에서 사용하는 서비스 IP (`10.43.21.0`)      |
| **LoadBalancer Ingress**                                                                      | 외부에서 접근할 수 있는 IP 주소들 (→ 아래 참고)            |
* → `192.168.164.130`, `192.168.164.131`: LoadBalancer가 노출한 **외부 접근 IP** (보통 VIP: Virtual IP) 
###### 🔌 포트 정보
| 항목             | 설명                                                                                    |
| -------------- | ------------------------------------------------------------------------------------- |
| **Port**       | 서비스가 노출하는 포트 (8080) - 클러스터 내부/외부 모두 사용 가능                                             |
| **TargetPort** | Pod 안의 컨테이너가 열어놓은 포트 (`80`)                                                           |
| **NodePort**   | 외부에서 노드 IP + 포트로 접근할 수 있도록 노출된 포트 (`30088`)                                           |
| **Endpoints**  | 현재 이 서비스가 연결하고 있는 Pod의 IP와 포트 (즉, 실제 nginx 컨테이너 위치)<br>`10.42.4.11:80` = Pod IP\:Port |
###### 🔁 트래픽 정책
| 항목                          | 설명                                             |
| --------------------------- | ---------------------------------------------- |
| **Session Affinity**        | 클라이언트 요청을 항상 같은 Pod에 보내는지 여부 (`None`: 무관하게 분산) |
| **External Traffic Policy** | 외부 트래픽 처리 방식 (`Cluster`: 모든 노드가 트래픽을 처리 가능)    |
| **Internal Traffic Policy** | 클러스터 내부 트래픽 처리 정책 (`Cluster`: 모든 노드에서 접근 가능)   |
###### 📅 이벤트 로그
* 이 항목은 서비스 생성/업데이트 시 일어난 이벤트를 보여줍니다:

| Type       | Reason                 | 설명    |
| ---------- | ---------------------- | -------------------- |
| **Normal** | `EnsuringLoadBalancer` | 서비스 타입이 `LoadBalancer`이므로 외부 로드밸런서를 생성하려는 시도 시작                                                                                              |
| **Normal** | `AppliedDaemonSet`     | LoadBalancer 서비스를 위해 특정 DaemonSet (예: `svclb-*`)이 생성됨<br>→ 클러스터에 LoadBalancer 컨트롤러(예: kube-vip, klipper-lb 등)가 동작 중임을 의미                     |
| **Normal** | `UpdatedLoadBalancer`  | LoadBalancer IP가 다음처럼 업데이트됨:<br>→ `[] → [192.168.164.130]`<br>→ 이후 `→ [192.168.164.130 192.168.164.131]`<br>→ VIP가 2개로 확장됨 (노드가 여러 개인 경우 가능) |



```bash
# <로드밸런서IP>:<Service Port>로 호출합니다.
curl 192.168.164.130:8080
```

```bash
# 💡 Tip: K3s v1.27+부터는 servicelb 대신 Gateway API를 점진적으로 도입 중, 따라서 svclb pod이 생성되지 않는다.
kubectl get pod
# NAME                   READY   STATUS    RESTARTS   AGE
# ...
# svclb-load-bal-5n2z8   1/1     Running   0          4m
# svclb-load-bal-svv8j   1/1     Running   0          4m
```

### 6.2.4 ExternalName
#### 🌐 ExternalName 서비스란?
##### ✅ 정의:
* ExternalName 서비스는 클러스터 내부에서 도메인 이름을 다른 외부 DNS 이름으로 매핑하는 DNS 별칭(alias) 역할만 합니다.
* 즉, 클러스터 내부에서 특정 이름으로 요청을 보내면, 그 요청은 외부의 실제 DNS 주소로 리디렉션됩니다.

##### 📦 작동 예시
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-google
spec:
  type: ExternalName
  externalName: www.google.com
```

##### 🎯 동작 설명:
* kubectl apply -f ... 후,
* 클러스터 내부의 Pod가 external-google.default.svc.cluster.local로 요청을 보내면,
* DNS 질의 결과는 www.google.com으로 변환됨 (실제 IP를 반환).
* Kubernetes는 프록시를 하지 않고, 클라이언트가 직접 외부로 연결합니다.

##### 🧠 언제 사용할까?
| 사용 시점                                          | 설명                               |
| ---------------------------------------------- | -------------------------------- |
| 클러스터 내부에서 외부 DNS 이름을 **내부 서비스 이름처럼 사용**하고 싶을 때 | 예: 외부 DB, 외부 API                 |
| **외부 서비스로의 DNS alias를 만들고 싶을 때**               | 외부 도메인을 내부 도메인처럼 호출              |
| 클러스터 외부에 있는 시스템을 **서비스처럼 접근**하고 싶을 때           | 예: `mysql.external.com` 같은 외부 DB |

##### ⚠️ 주요 특징과 주의사항
| 항목                | 설명                                     |
| ----------------- | -------------------------------------- |
| 🔁 프록시 없음         | kube-proxy가 트래픽을 처리하지 않음 (단순 DNS 매핑)   |
| 🌐 클러스터 외부 DNS 사용 | 실제 DNS lookup을 통해 외부 IP로 접속            |
| 🔐 보안 고려 필요       | 클러스터에서 외부 서비스에 직접 연결되므로 TLS 등 보안 처리 필요 |
| ❌ selector 없음     | 이 서비스는 `selector`가 필요하지 않음             |
| 📛 port 설정 안 함    | 포트 매핑도 필요 없음 (단순 도메인 이름만 리다이렉트)        |

##### 📌 예시: 외부 MySQL에 접속
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-mysql
spec:
  type: ExternalName
  externalName: db.example.com
```
* 클러스터 내 앱은 external-mysql.default.svc.cluster.local에 접속
* 실제로는 DNS 질의 결과 db.example.com으로 리디렉션됨

##### 🎯 비교: 다른 서비스 타입과 차이
| 타입               | 목적                | 외부 노출     | DNS 매핑 | kube-proxy |
| ---------------- | ----------------- | --------- | ------ | ---------- |
| ClusterIP        | 클러스터 내부 통신        | ❌         | ✅      | ✅          |
| NodePort         | 노드 포트로 외부 접근      | ✅         | ✅      | ✅          |
| LoadBalancer     | 클라우드 LB로 외부 노출    | ✅         | ✅      | ✅          |
| **ExternalName** | **외부 도메인으로 리디렉션** | ❌ (직접 연결) | ✅      | ❌          |


##### ✅ 요약
* ExternalName은 Kubernetes 서비스 이름 → 외부 도메인 이름으로 DNS alias 역할을 합니다.
* 실제 트래픽은 프록시하지 않고, 클러스터 내부에서 직접 외부로 연결됩니다.
* 내부 서비스처럼 외부 API나 DB 등을 연결하고 싶을 때 간단히 사용할 수 있는 유용한 기능입니다.


##### 📌 실습

```bash
vi K8S/CH05/external.yaml
```

```yaml
# external.yaml
# 클러스터내에 google-svc 별칭을 이용하여 google.com에 접속
apiVersion: v1
kind: Service
metadata:
  name: google-svc  # 별칭
spec:
  type: ExternalName
  externalName: google.com  # 외부 DNS
```

```bash
kubectl apply -f K8S/CH05/external.yaml
# service/google-svc created

kubectl get svc

# 클러스터 안에서 curl 컨테이너를 실행하고,
# 서비스 이름 google-svc를 통해 외부 도메인(예: google.com)에 접속 테스트
kubectl run call-google --image curlimages/curl \
              --  curl -s -H "Host: google.com" google-svc
# pod/call-google created

# call-google의 status가 Completed 와 CrashLoopBackOff  반복
# 이 명령은 curl 명령을 실행하는 단일 컨테이너 Pod를 생성합니다.
# curl 요청이 끝나면 컨테이너는 정상 종료(= Completed) 합니다.
# 그런데 Kubernetes는 기본적으로 Pod의 restartPolicy를 Always로 설정합니다.
# 👉 그래서 컨테이너가 정상 종료되었음에도 불구하고, kubelet이 다시 시작하려고 시도합니다
```

##### 🔍 라인별 설명
| 구성 요소                     | 설명                                                         |
| ------------------------- | ---------------------------------------------------------- |
| `kubectl run call-google` | 이름이 `call-google`인 Pod(또는 일시적인 Job)를 생성                    |
| `--image=curlimages/curl` | `curl` 명령어만 포함된 경량 이미지 사용                                  |
| `--`                      | 이 이후부터는 컨테이너 내에서 실행할 **명령어**를 의미                           |
| `curl -s`                 | `curl` 명령어, `-s`는 **silent 모드** (진행 상태 미출력)                |
| `-H "Host: google.com"`   | HTTP 요청 시 Host 헤더를 `google.com`으로 설정                       |
| `google-svc`              | 요청 대상 → Kubernetes 서비스 이름 (`ExternalName` 서비스로 등록되어 있어야 함) |

```bash
kubectl logs call-google
# <HTML><HEAD><meta http-equiv="content-type" content="text/..">
# <TITLE>301 Moved</TITLE></HEAD><BODY>
# <H1>301 Moved</H1>
# ...
```

###### 참고할 자료
* 네트워크 구조: https://kubernetes.io/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model
* 쿠버네티스 네트워킹 이해하기: https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01

<img src="./images/kubernets install/쿠버네티스 네트워크 구조.png" width="600" height="400">
<img src="./images/kubernets install/쿠버네티스 네트워크 묘델.webp" width="600" height="400">


### Clean up

```bash
kubectl delete svc --all
kubectl delete pod --all
```